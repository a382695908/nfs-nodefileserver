# nfs-nodefileserver

## 一个基于node.js的文件服务，可作为中小型项目的文件服务，目前完成了，文件上传，文件分片上传，文件合并等功能，数据库方面用的是mysql，前端的上传插件调用的是百度的webuploader当然你也可以使用其他的三方插件或是直接表单提交文件都是OK的，具体的API文件将在一周内开放，目前只是测试版本还有大概15%的方法尚未完成，不影响基础的使用流程。

## 本项目是本人用作学习node.js的项目肯定有很多不足之处，所以希望尽早开放出来多听一些意见来改进学习，此外不接受任何负面的评论，项目将采用mpl开源协议，只求使用者声明该部分是使用该项目的即可，使用用户多的话会开一个单独的站点来说明，同时该项目还有个.net版本的若有需要可以联系作者。QQ群:192321831，QQ:290341190（暂时开放）欢迎加好友一起讨论。更多详情也可以关注作者的博客：www.playworld.top

## 开始使用

### 1、首先你要确保你的服务器已经包含了Node.js的运行环境
### 2、在项目路径下public/database/nfs_nodefileserver.sql,是Mysql的创建数据库的SQL语句，执行完成后在根目录下的nfsconfig.json中进行数据库配置，如下：
```json
"openDatabase": true,
 "poolContent":{
    "host"     : "localhost",
    "user"     : "root",
    "password" : "root",
    "port": "3306",
    "database": "nfs_nodefileserver"
  },
```
### 当openDatabase:false时回去读取xmlpath，在xmlpath处进行xml格式的文件存储，该功能暂时在部分功能下会失效（还没完成）

### 3、之后就可以启动项目并访问端口号3000来进行整个文件服务的测试

### 4、进入页面后，只点击上传按钮选择文件后即完成上传功能，此时会在nfsconfig.json配置文件中设置的filePath的位置将上传的文件保存至此处。在返回内容中拷贝fileid(文件id)至需下载的FileID的input框中点击下载按钮即完成了整个上传下载的流程。
![](http://97.64.36.122:886/wp-content/uploads/2018/01/QQ截图20180117131536.png)

### 5、由于中途加入了阿里云文件备份的SDK所以，ws报了一个javascript版本不正确，在修改为es6后成功运行，这个问题在正式环境下还没有进行过测试，如果出现相同问题可以注释掉ali云备份相关代码。

## API使用文档

### 以下所有的接口仅供参考请以router/index.js下的方法为准，因为整个项目仍处于编写阶段存在，API文档未及时更新的可能性。但是基本不影响整个项目的使用

### 1. nfs提供了一个基础的文件上传的方法：uploadsimplefile。使用者将文件以post请求的方式进行提交，允许使用三方插件，本项目就是以webuploader作为基础的测试插件来进行使用的。参数有md5（string型，当md5为空时会自动生成一个uuid代替）,uploadSourceType（INT型，上传来源，如果项目用到的地方多的话可以传该参数），fileType(INT型，文件类型，或者说是用途，标签之类的：娱乐、游戏、视频等)。返回结果如上图所示。

### 2. nfs提供了一个文件分片上传的方法：uploadchunkfile。该方法是基于前端已经进行了分片处理，据我所知webuploader,uploadfy等都是包含分片方法的。在测试页面中点击开启分片后，文件就以分片的形式进行上传，分片大小为2M每个。由于大文件的md5值计算较慢，在短时间内会提供本人之前用的一种方式。

### 3. nfs提供了一个文件下载的方法：downLoadFiles。根据提交请求的参数fileIds（多个文件id以英文的逗号','隔开），isZip(string型，"true"或者"false"),当下载单个文件且isZip="true"时将进行压缩下载，当请求下载多个文件时，自动后台压缩后下载。目前isZip还未完成所以只支持单个文件的上传和下载。下图为多个文件的压缩下载：
![](http://97.64.36.122:886/wp-content/uploads/2018/01/QQ截图20180119092558.png)
### 4. nfs提供了一个获取文件信息的方法：getFilesInfo。该方法为get方法，根据提交请求的参数fileIds（多个文件id以英文的逗号','隔开），返回对应的文件信息
![](http://97.64.36.122:886/wp-content/uploads/2018/01/QQ截图20180117152804.png)

### 5. nfs提供了一个获取分片信息的方法：getChunkInfo。该方法为get方法，根据提交请求的参数fileIds（多个分片id以英文的逗号','隔开），返回对应的分片信息

### 6. nfs提供了一个验证文件是否存在的方法：fileExist。该方法为get方法，根据提交请求的参数md5，验证文件是否存在（包括物理路径下是否存在），主要是用于内部文件验证使用所以暂不开放一次性请求多个md5的方法。

### 7. nfs提供了一个验证分片是否存在的方法：chunkExist。该方法为get方法，根据提交请求的参数md5和chunk，验证分片是否存在（包括物理路径下是否存在），主要是用于内部分片验证使用所以暂不开放一次性请求多个md5的方法。

### 8. nfs提供了一个文件合并的方法：mergeFile。该方法为get方法，根据提交请求的参数md5值，去数据库找到对应路径后进行一些必要的验证（文件数之类的）后进行文件合并。合并完成后会在n_file_info表中新插入一条该合并文件的数据并返回。注：可能与设计理念有关，每个文件有一个唯一的md5值，只要md5值相同的文件，我便认为是同一个文件剩余的文件信息是入库的。

### 9. nfs提供了一个文件删除的方法：deleteFiles。该方法为get方法，根据提交请求的参数fileIds（多个文件id以英文的逗号','隔开），当配置文件canPhysicalDelete（是否允许物理删除）:false时只进行数据库status的置为1，否则会去文件的对应位置进行删除操作。
![](http://97.64.36.122:886/wp-content/uploads/2018/01/QQ截图20180117164504.png)
### 10. nfs提供了一个分片删除的方法：deleteChunks。该方法为get方法，根据提交请求的参数fileIds（多个分片id以英文的逗号','隔开），当配置文件canPhysicalDelete（是否允许物理删除）:false时只进行数据库status的置为1，否则会去文件的对应位置进行删除操作。

### 11. 整个项目还在加快进度完成中，所有方法具体可以参考router中的index.js或者nfs_module中的nfs里面都包含有详细的中文说明，若方法中尚未包含具体代码即尚未完成。

### 12. nfs中大部分的方法都是采用callback回调来返回结果的。并且编写了一个result的返回结果类。result总共有三个属性sate,message,data并且提供了三个方法result.set(state,message,data),result.error(message),result.success(message)。目前大部分的get方法的返回结果都是以这样的格式进行返回的。
![](http://97.64.36.122:886/wp-content/uploads/2018/01/QQ%E6%88%AA%E5%9B%BE20180117140519.png)

### 13. nfs在设计过程中提供了文件备份功能，主要是讲文件备份至七牛云和阿里的oss对象存储。目前提供了fileBackUpByQiNiu和fileBackUpByAli两个文件备份至云上的测试方法。但对于项目的需求来说尚未完成，仍在代码编写测试阶段。

# 以下是将要解决或已经结局的部分

##可能存在的问题：module的安装在当前项目中无法安装，所有后续安装的module都是从另一个项目中拷过来的。所以暂时package.json的是没有包括的，等后续手动加进去不知道会不会有问题
##文件下载，比如下载请求数大于5时拒绝后续请求，日志系统，文件放满，跨域，分布式
##整个错误的回调机制
##在文件分片上传过程中若发现该文件存在于数据库表中，而该目录地址下不存在该文件，暂时验证时为文件不存在，即可以上传。整个逻辑肯定会出现bug，不过当不存在人为对文件或
数据库进行操作时不会出现问题。
##暂时前端页面不支持多个文件一起分片上传，因为我还做不到根据文件传md5值，现在的md5是一样传上来的。
##基于安全考虑一开始上传的文件就是不带后缀的，这次连文件名中也不再包含任何文件信息以"chunk-chunks"的格式来存放。
##参数一致性，因为项目是本人一人完成的，所以从前端页面到后端代码及数据库，牵一发动全身，为了开发的快速，有部分分片的参数里仍存在fileIds的字段
##存在某种需求，要求下载文件以完整目录格式压缩后下载

嗯~本人接受支付宝捐赠：290341190@qq.com（请填写备注，将用于之后站点建设时的感谢部分。）

## 本项目所有权归作者：方一行 所有。开源协议暂未填写完成故此申明，作为开源项目禁止一切二次开发后单独出售的情况，所有引用本项目的都需有明确注释声明该项目的所有者，所有二次开发再开源的都需要知会作者一声，作者不接受任何由该项目引发的负面责任以上。
