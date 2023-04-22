# Bilibili视频评论爬虫

* 能**批量**爬取B站多个视频的评论，使用Selenium而非B站api，能爬取到更**全面**的数据。
* 能够**断点续爬**，可以随时关闭程序，等到有网络时再继续运行。
* 遇到错误**自动重试**，非常省心，可以让它自己爬一整晚。

#### 好用的话记得给个star！

## 功能
1. **能爬取二级评论 爬取字段更丰富**。输出文件将包含以下字段：一级评论计数、隶属关系（一级/二级评论）、被评论者昵称、被评论者 ID、评论者昵称、评论者用户 ID、评论内容、发布时间、点赞数。     
2. **批量爬取多个视频的评论**：把要爬取的网址写进 video_list.txt 即可，每个视频的评论都会输出一个以视频ID命名的 CSV 文件。
3. **只需一次登录**：一次手动登录后，程序会记下你的 cookies ，存放在代码同级文件夹 cookies.pkl 内。在 cookies.pkl 被手动删除前，程序每次运行都会自动登录（cookies失效后请删除cookies.pkl文件）。
4. **断点续爬**：中断后，爬虫可以根据 progress.txt 文件中的进度继续爬取。写入到一半的 CSV 文件也会继续写入。

### 关于断点续爬与progress
* 断点续爬功能依托progress.txt记录实现：程序运行时，如果代码同级文件夹内存在progress.txt文件，它会读取其中进度；如果没有，则自动创建。
* 如果想要从头开始爬取，只需删除 progress.txt 文件即可。
* 如果想要修改爬虫任务，跳过某些视频/一级评论/二级评论页，建议**直接修改progress.txt文件**。    
（例如，有一个视频爬取失败，想要跳过它，直接在progress中，把video_count加1即可）        
* progress含义：    
第{video_count}个视频已完成爬取。    
第{video_count + 1}个视频中，第{first_comment_index}个一级评论的，二级评论第{sub_page}页已完成爬取。    
"write_parent"为1指当前一级评论已写入，为0指当前一级评论尚未写入。     
示例如右：`{"video_count": 1, "first_comment_index": 15, "sub_page": 114, "write_parent": 1}`            
注意："video_count""first_comment_index""sub_page"三个值全部是从0开始的,"write_parent"取值为0或1。    

## 安装
1. 安装 Python 3。
2. 安装所需的库。在命令行中输入以下命令：pip install selenium beautifulsoup4 webdriver-manager

## 使用
1. 将要爬取评论的视频 URL 列表放入名为 video_list.txt 的文件中，每行一个 URL。
2. 参数设定
  * 若要修改最大滚动次数（默认45次，预计最多爬取到900条一级评论），请在代码中修改参数MAX_SCROLL_COUNT的值。注意，滚动次数过多，加载的数据过大，网页可能会因内存占用过大而崩溃。
  * 若要设定最大二级评论页码数（默认为150页），请在代码中修改参数max_sub_pages的值（若想无限制，请设为max_sub_pages = None）。建议设定一个上限以减少内存占用，避免页面崩溃。
4. 运行代码：python Bilicomment.py（或pycharm等软件打开运行）。代码使用selenium爬取数据。
5. 根据看到"请登录，登录成功跳转后，按回车键继续..."提示后，请登录 Bilibili。登录成功并跳转后，回到代码，按回车键继续。
6. 等待爬取完成。每个视频的评论数据将保存到以视频 ID 命名的 CSV 文件中， CSV 文件位于代码文件同级目录下。
7. 输出的 CSV 文件将包括以下列：'一级评论计数', '隶属关系'（一级评论/二级评论）, '被评论者昵称'（如果是一级评论，则为“up主”）, '被评论者ID'（如果是一级评论，则为“up主”）, '昵称', '用户ID', '评论内容', '发布时间', '点赞数'。        
![爬取字段示例](/image/output_sample.png)
7. 输出的 CSV 文件是utf-8编码，若**乱码**，请检查编码格式（可以先用记事本打开查看）。
8. 如果有视频因为错误被跳过，将会被记录在代码同级文件夹下的video_errorlist.txt中。

## 注意事项
### 关于输出结果
1. 因为B站存在评论数虚标，部分评论可能被封禁或隐藏，所以爬取到的评论数量通常小于标称数量。只要自己在网页中不断下滑看到的最后几条评论和代码爬取的最后几条数据相符合，所有评论就已被完整爬取了。
2. 如果是一级评论，则'被评论者昵称'和'被评论者ID'都会写上"up主"几个字。这不是爬取的结果，而是程序自行写入的文字。

### 关于可能的问题
1. 用Excel打开 CSV 文件查看时，可能会发现有些单元格报错显示"$NAME?"，这是由于这个单元格的内容是以"-"符号开头的，例如昵称"-Ghauster"。
2. 如果代码报错Permission denied，请查看是否有别的进程占用了正在写入中的 CSV 文件或 progress.txt 文件（比如，文件被我自己打开了），检查是否有写入权限。还不行，可以尝试**以管理员身份**运行代码（遇到PermissionError，都可以尝试以管理员身份运行来解决）。
3. 爬取超大评论量的热门视频时，网页可能会因为内存不足而崩溃。如果发生这种情况，程序会在一定时间后**自动重启浏览器断点续爬**。但是如果网页都还没有滚动到底全部加载完、都还没有开始爬，就内存不足了，那无论自动重试多少次都会重复出现网页崩溃的问题，此时建议限制最大滚动次数。
4. 在使用selenium + chrome浏览器爬取数据时，如果该视频评论量过大，selenium模拟浏览器会产生大量的临时文件。目前，程序将缓存存储在代码文件所在目录中，重试续爬前我们可以自行删除。
5. 运行爬虫时，请不要开着全局模式网络代理。网络**过于卡顿和不稳定**都有可能引发报错。
6. 如果程序长时间没有动静（控制台长时间没有打印当前进度），就重启程序吧，它会断点续爬的。
