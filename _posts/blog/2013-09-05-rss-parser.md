---
published: false
---

广为大家使用的Java开源RSS解析工具大致有`rsslib4j`和`ROME`，对于这些feed parser，将笼统的给出一些对比，并根据需求进行选择。本人选用了ROME进行了rss解析，并给出简单的使用示例代码。

其中rsslib4j只支持rss格式，ROME既支持rss也支持atom,兼容性较好。（占位编辑）

    package util;
    
    import java.net.URL;
    import java.util.List;
    
    import com.sun.syndication.feed.synd.SyndCategory;
    import com.sun.syndication.feed.synd.SyndContent;
    import com.sun.syndication.feed.synd.SyndEnclosure;
    import com.sun.syndication.feed.synd.SyndEntry;
    import com.sun.syndication.feed.synd.SyndFeed;
    import com.sun.syndication.io.SyndFeedInput;
    import com.sun.syndication.io.XmlReader;
    
    public class RssParser {
        
        @SuppressWarnings("rawtypes")
        public void parseRss(String rss) {
    
            // String rss =
            // "http://news.baidu.com/n?cmd=1&class=civilnews&tn=rss&sub=0]http://news.baidu.com/n?cmd=1&class=civilnews&tn=rss&sub=0";
    
    
            try {
                URL url = new URL(rss);
                // 读取Rss源
                XmlReader reader = new XmlReader(url);
                //XmlDocument 
    
                System.out.println("Rss源的编码格式为：" + reader.getEncoding());
                SyndFeedInput input = new SyndFeedInput();
                // 得到SyndFeed对象，即得到Rss源里的所有信息
                SyndFeed feed = input.build(reader);
    
                // System.out.println(feed);
    
                // 得到Rss新闻中子项列表
                List entries = feed.getEntries();
                // 循环得到每个子项信息
                for (int i = 0; i < entries.size()&&i<5; i++) {
                    SyndEntry entry = (SyndEntry) entries.get(i);
    
                    // 标题、连接地址、标题简介、时间是一个Rss源项最基本的组成部分
                    System.out.println("标题：" + entry.getTitle());
                    System.out.println("连接地址：" + entry.getLink());
                    SyndContent description = entry.getDescription();
                    System.out.println("标题简介：" + description.getValue());
                    System.out.println("发布时间：" + entry.getPublishedDate());
    
                    // 以下是Rss源可选部分
                    System.out.println("标题的作者：" + entry.getAuthor());
    
                    // 此标题所属的范畴
                    List categoryList = entry.getCategories();
                    if (categoryList != null) {
                        for (int m = 0; m < categoryList.size(); m++) {
                            SyndCategory category = (SyndCategory) categoryList
                                    .get(m);
                            System.out.println("此标题所属的范畴：" + category.getName());
                        }
                    }
    
                    // 得到流媒体播放文件的信息列表
                    List enclosureList = entry.getEnclosures();
                    if (enclosureList != null) {
                        for (int n = 0; n < enclosureList.size(); n++) {
                            SyndEnclosure enclosure = (SyndEnclosure) enclosureList.get(n);
                            System.out.println("流媒体播放文件：" + entry.getEnclosures());
                        }
                    }
                    System.out.println();
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
    
        }
    }