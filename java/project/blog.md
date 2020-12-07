[TOC]
# 登录模块
## 实现方法
基于Redis实现了单点登录。用Redis模拟session。每次访问会随机产生一个sessionid,并将该id存入到redis中。同时登录后，会将该id以cookie的形式返回给客户端。下次登录时，会利用基于aop的环绕通知实现的LoginAspect检查Redis是否存了这个cookie,如果存了，则直接允许登录
## 流程图
![](https://gitee.com/zacharytse/image/raw/master/img/das.png)

## 实现过程中遇到的问题
在处理cookie跨域问题时，错误地将项目部署地址的url设置为了cookie的domain
# 搜索模块
## 分词模块
模仿jieba实现
### 实现方法
1. 先用字典树将词典中的单词全部读入完成初始化。
2. 对于输入的句子，会先进行清洗，将非中文的字符提取出来单独保存。
3. 扫描整个句子的子集，并将这些子集构造成全DAG图(对于词不在字典中的情况还没有做处理)
4. 做划分时，通过max{dp[x] + score(subStr)}选出最长路径，该最长路径即为最后的划分结果

第三步代码如下:
```java{.line-numbers}
private List<Integer>[] buildGraph(String sentence) {
        List<Integer>[] graph = new List[sentence.length()];
        for (int i = 0; i < sentence.length(); ++i) {
            for (int j = i + 1; j <= sentence.length(); ++j) {
                String word = sentence.substring(i, j);
                if (scanner.hasThisWord(word)) {
                    if (graph[i] != null) {
                        graph[i].add(j);
                    } else {
                        List<Integer> list = new ArrayList<>();
                        list.add(j);
                        graph[i] = list;
                    }
                } else {
                    //处理不在词典中的情况,先暂时空着
                }
            }
        }
        return graph;
    }
```
第4步如下:
```java{.line-numbers}
public List<String> divide(String word) {
        List<Pair<Character, Integer>> specialData = cleanData(word);
        List<Integer>[] graph = buildGraph(word);
        double[] dp = new double[word.length() + 1];
        String[] ans = new String[word.length()];
        for (Pair<Character, Integer> data : specialData) {
            ans[data.getValue()] = String.valueOf(data.getKey());
        }
        int n = word.length();
        for (int idx = n - 1; idx >= 0; --idx) {
            String maxWord = "";
            List<Integer> list = graph[idx];
            if (list == null) {
                continue;
            }
            double maxScore = -Double.MAX_VALUE;
            for (Integer x : list) {
                String curWord = word.substring(idx, x);
                double score = scanner.getScore(curWord) + dp[x];
                if (Double.compare(score, maxScore) > 0) {
                    maxScore = score;
                    maxWord = curWord;
                }
            }
            dp[idx] = maxScore;
            ans[idx] = maxWord;
        }
        return handleResult(ans);
    }
```
## 构建索引
对于每个单词，都采取倒排索引，即单词本身作为索引，而单词所在的Document,field均作为value进行存储

添加索引的代码如下:
```java{.line-numbers}
private void add(Map<String, IndexTerm> map,
                     IndexTerm term) {
        synchronized (map) {
            IndexTerm storedTerm = map.get(term.getVal());
            if (storedTerm == null) {
                map.put(term.getVal(), term);
            } else {
                long docId = term.getDocIds().get(0);
                int fieldId = term.findAllFieldIdByDocumentId(docId).get(0);
                int startId = term.findAllStartIdByDocumentIdAndFieldId(docId, fieldId).get(0);
                String val = term.getVal();
                storedTerm.addNewWord(val, docId, fieldId, startId);
            }
        }
    }
```
其中IndexTerm是索引的最小单位
```java{.line-numbers}
public class IndexTerm {

    private Logger logger = LoggerFactory.getLogger(getClass());

    private String val;
    //包含的文档id
    private List<Long> docIds;
    //包含的field id
    //格式<docId:[fieldId....]>
    private Map<Long, List<Integer>> fieldIds;
    //在所有的field中的出现的位置
    //格式<docId:<Field:[startId...]>>
    private Map<Long, Map<Integer, List<Integer>>> startIds;
    //在所有文档中出现的频率
    //格式<docId:value_freq>
    private Map<Long, Integer> freq;
    //剩余代码省略
}
```
### 构建方法
构建索引是通过IndexWriter实现
```java{.line-numbers}
/**
 * 写入索引数据
 */
public class IndexWriter {

    private DocumentsWriter writer;

    public IndexWriter(Directory directory,IndexWriterConfig config) {
        writer = new DocumentsWriter(directory,config);
    }

    /**
     * 将document加入到待写入索引队列中
     * @param doc
     */
    public void addDocument(Document doc) {
        writer.add(doc);
    }

    /**
     * 只有调用该方法，才会真正把索引写入进去
     */
    public void commit() {
        writer.commit();
    }
}
```
而IndexWriter具体实现是交由DocumentsWriter。DocumentsWriter内部有一个newFixedThreadPool类型的线程池。
借助add方法可以把Document加入到任务队列中，只有调用commit方法，才会执行任务队列中的任务。
commit()方法执行时，会阻塞主线程，直到线程池中的所有任务完成。
这种阻塞功能的实现是通过CountDownLatch完成

```java{.line-numbers}
    public void commit() {
        try {
            if (!hasTaskNeedExecuted()) {
                return;
            }
            latch = new CountDownLatch(tasks.size());
            executeAllTask();
            latch.await();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
//....
        @Override
        public void run() {
            List<Field> fields = doc.getFields();
            TermDividerAdapter adapter = new TermDividerAdapter(divider);
            for (Field field : fields) {
                addIndex(field.getFieldId(), adapter.divide(field.getName()), true);
                addIndex(field.getFieldId(), adapter.divide(field.getContent()), false);
            }
            latch.countDown();
        }
```
这里为了不改变分词模块接口，使用适配器重新对分词模块的接口进行包装。

> 分成add方法和commit方法的原因

如果直接add添加的话，可能会出现子线程还未执行添加操作，而主线程此时执行了查询操作，主线程此时显然会得到null的结果。所以通过commit方法阻塞主线程，保证所有任务全部执行完成。此时可以保证主线程查询结果的正确性。

### 解决搜索关键字和索引不匹配的问题
有可能搜索的关键字在索引中完全不存在，但此时返回null感觉也不太合适。所以采用EDR算法，在查找索引的时候判断关键字和当前索引的相似度是否高到某个阈值，如果高于某个阈值，则可以认为两者就是相等的，此时把该索引返回即可

EDR算法实现如下:
```java{.line-numbers}
private int getSimilarityByEDR(String word1, String word2) {
        int n = word1.length();
        int m = word2.length();
        int[][] edr = new int[n][m];
        edr[0][0] = 0;
        for(int i = 0; i < m; ++i) {
            edr[0][i] = i;
        }
        for (int i = 0; i < n; ++i) {
            edr[i][0] = i;
        }
        for (int i = 1; i < n; ++i) {
            for (int j = 1; j < m; ++j) {
                int edit = word1.charAt(i) == word2.charAt(j) ? 0 : 1;
                edr[i][j] = Math.min(edr[i - 1][j - 1] + edit,
                        Math.min(edr[i][j - 1] + 1, edr[i - 1][j] + 1));
            }
        }
        return edr[n - 1][m - 1];
    }
```

查找索引的过程
```java{.line-numbers}
private synchronized IndexTerm doFindTerm(String word, Map<String,IndexTerm> map) {
        for (Map.Entry<String, IndexTerm> entry : map.entrySet()) {
            if (config.isSimilar(word, entry.getKey())) {
                return entry.getValue();
            }
        }
        return null;
    }
```

## 具体Document的查找
这里定义了IDocumentQuery接口。底层的索引只记录了document的id，并没有存储具体的数据。在查找到具体的id后，可以通过IDocumentQuery接口查找具体的数据。但该接口并没有给出具体的实现。在使用搜索模块时，可以自己定义一个实现该接口的类。

```java{.line-numbers}
public interface IDocumentQuery {

    List<Document> findAllDocumentsById(List<Long> ids);

    Document findDocumentById(long id);
}
```
具体调用如下:

```java{.line-numbers}
private DocumentWrapper wrapperDocument(Long docId, IndexTerm term) {
        Document doc = query.findDocumentById(docId);
        Map<Integer, List<Integer>> pos = term.getStartIds(docId);
        DocumentWrapper wrapper = new DocumentWrapper(doc, pos);
        return wrapper;
    }
```

这里提供一个测试用的实现类

```java{.line-numbers}
public class TestQueryImpl implements IDocumentQuery {

    private List<Document> documents;

    public TestQueryImpl() {
        documents = new ArrayList<>();
        Document doc = new Document();
        doc.setDocId(0);
        doc.add(new TextField("多少分呢", "的范围"));
        doc.add(new TextField("发生的", "非完全阿斯顿"));
        documents.add(doc);
        doc = new Document();
        doc.setDocId(1);
        doc.add(new TextField("大大双", "分为范围是"));
        doc.add(new TextField("范文芳", "分为范围地方v我热夫"));
        documents.add(doc);
    }

    @Override
    public List<Document> findAllDocumentsById(List<Long> ids) {
        return documents;
    }

    @Override
    public Document findDocumentById(long id) {
        final Document document = documents.get((int) id);
        return document;
    }
    //方便测试的接口
    public List<Document> getDocuments() {
        return documents;
    }
}
```
测试代码如下:
```java{.line-numbers}
    @Test
    public void testDocumentSearch() {
        TermDividerSingleton.getDivider();
        TestQueryImpl query = new TestQueryImpl();
        List<Document> documents = query.getDocuments();
        DocumentSearch search = new DocumentSearch(query);
        for(Document document : documents) {
            search.addDocument(document);
        }
        search.commit();
        List<DocumentWrapper> wrappers = search.searchContent("范");
        for(DocumentWrapper wrapper : wrappers) {
            List<Field> fields = wrapper.getFields();
            for(Field field : fields) {
                System.out.println(field.getName());
            }
        }
    }
```
## 待实现的功能
- 输入搜索语句后，还没有对该语句做关键词的提取，目前是把整个搜索语句当作完整的词语去进行匹配。
- 没有加入索引的更新，删除(IndexUpdate,IndexDelete)
- 没有加入缓存模块(最好也是弄成可扩展的搜索缓存,即可以扩展成各种各样的缓存)

# 评论模块
![](https://gitee.com/zacharytse/image/raw/master/img/20201206213642.png)
## 数据库创建
comment表结构如下
![](https://gitee.com/zacharytse/image/raw/master/img/20201206204809.png)

在更新用户名时，通过触发器对comment表中的targetName进行更新，更新代码如下
```sql
use blog;
delimiter //
DROP TRIGGER IF EXISTS `updateUsername`//
CREATE TRIGGER `updateUsername` AFTER UPDATE ON `users`
FOR EACH ROW BEGIN
UPDATE `comment` SET `comment`.`targetName` = NEW.`username` WHERE  `comment`.`targetId` = OLD.`id`;
END
//
```
