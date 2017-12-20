    List<T>的各种排序方法

排序自然就会想到用Sort方法，看看List<T>的Sort方法各个重载版本：

    public void Sort(); //1
    public void Sort(Comparison<T> comparison); //2
    public void Sort(IComparer<T> comparer);    //3
    public void Sort(int index, int count, IComparer<T> comparer);

---------------------------------------------------------------------------------------------------

1：Sort()方法,摘要：使用默认比较器对整个 System.Collections.Generic.List<T> 中的元素进行排序,
    这里的默认比较器就是指Comparer<T>.Default。要使用默认比较器来排序，则类必须实现IComparable<T>接口，
    排序的时候会调用接口的CompareTo方法。
    接下来，就定义一个测试类Article，实现IComparable<Article>接口，排序先按SortIndex排序，再按Comments排序。
    类的定义如下：
    public class Article : IComparable<Article>
    {
        public string Title { get; set; }
        public int Comments { get; set; }
        public int SortIndex { get; set; }

        public override string ToString()
        {
            return string.Format("文章：{0}，评论次数:{1}", this.Title, this.Comments);
        }
        
        public int CompareTo(Article other)
        {
            if (other == null)
                return 1;
            int value = this.SortIndex - other.SortIndex;
            if (value == 0)
                value = this.Comments - other.Comments;
            return value;
        }
    }

    定义一个获取100万的Article集合的方法，Article的SortIndex，Comments都是随机生成的。实现代码如下：
        private static List<Article> GetArticleList()
        {
            List<Article> source = new List<Article>();
            Article article = null;
            var random = new Random(DateTime.Now.Millisecond);
            for (int i = 1000000; i > 0; i--)
            {
                article = new Article()
                {
                    Title = "文章" + i.ToString(),
                    Comments = random.Next(),
                    SortIndex = random.Next()
                };
                source.Add(article);
            }
            return source;
        }

    执行Sort方法排序：
        private static void SortByDefaultComparer()
        {
            List<Article> list = GetArticleList();
            list.Sort();
        }
        
-----------------------------------------------------------------------------------------------------

2：使用Comparison委托
    Comparison委托的定义如下：public delegate int Comparison<in T>(T x, T y);
    使用委托，可以传递一个与委托签名相同的函数，可以使用匿名委托，还可以用Lambda表达式：
    首先使用第一种方法:定义一个ArticleComparison类，在里面定义一个Compare方法，
    函数签名与Comparison委托保持一致。实现代码如下：
    public class ArticleComparison
    {
        public int Compare(Article x, Article y)
        {
            if (x == null)
            {
                if (y == null)
                    return 0;
                else
                    return -1;
            }
            else
            {
                if (y == null)
                {
                    return 1;
                }
                else
                {
                    int value = x.SortIndex.CompareTo(y.SortIndex);
                    if (value == 0)
                        value = x.Comments.CompareTo(y.Comments);
                    return value;
                }
            }
        }
    }

    //方法调用
    private static void SortByComparison()
        {
            List<Article> list = GetArticleList();
            list.Sort(new Comparison<Article>(new ArticleComparison().Compare));
        }

    接下来，使用Lambda表达式，实现代码如下：
    private static void SortByLambda()
        {
            List<Article> list = GetArticleList();
            list.Sort((x, y) =>
            {
                int value = x.SortIndex.CompareTo(y.SortIndex);
                if (value == 0)
                    value = x.Comments.CompareTo(y.Comments);
                return value;
            });
        }

---------------------------------------------------------------------------------------

3：使用自定义的IComparer来排序，这里自定义一个ArticleCompare类：实现代码如下：
    public class ArticleCompare : IComparer<Article>
    {
        public int Compare(Article x, Article y)
        {
            if (x == null)
            {
                if (y == null)
                    return 0;
                else
                    return -1;
            }
            else
            {
                if (y == null)
                {
                    return 1;
                }
                else
                {
                    int value = x.SortIndex.CompareTo(y.SortIndex);
                    if (value == 0)
                        value = x.Comments.CompareTo(y.Comments);
                    return value;
                }
            }
        }
    }
    //方法调用
    private static void SortByCustomerComparer()
        {
            List<Article> list = GetArticleList();
            list.Sort(new ArticleCompare());
        }

-------------------------------------------------------------------------------------------

***：使用LINQ排序，实现代码如下：
        private static void SortByLinq()
        {
            List<Article> list = GetArticleList();
            var sortedList =
               (from a in list
                orderby a.SortIndex, a.Comments
                select a).ToList();
        }

-------------------------------------------------------------------------------------------

***：使用扩展方法OrderBy，OrderByDescending，实现代码如下：
        private static void SortByExtensionMethod()
        {
            List<Article> list = GetArticleList();
            var sortedList = list.OrderBy(a => a.SortIndex).ThenBy(a => a.Comments);
            sortedList.ToList(); //这个时候会排序
        }
