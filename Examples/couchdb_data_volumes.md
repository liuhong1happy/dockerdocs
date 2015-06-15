# Docker化CouchDB Service #

本例子，使用数据卷在两个CounchDB容器中来共享相同的数据。这可能被使用作为热门更新，测试不同CouchDB版本在相同的数据上。

## 创建第一个数据库 ##

我们标记`/var/lib/couchdb`作为数据卷

    $ COUCH1=$(sudo docker run -d -p 5984-v /var/lib/couchdb shykes/couchdb:2013-05-03)

在第一个数据库中添加数据

我们将确保你的docker host是localhost能访问的。如果不行，替换localhost为你Docker host的公有IP。

    $ HOST=localhost
    
    $ URL="http://$HOST:$(sudo docker port $COUCH1 5984 | grep -o '[1-9][0-9]*$')/_utils/"
    
    $ echo "Navigate to $URL in your browser, and use the couch interface to add data"

创建第二个数据库

    $ COUCH2=$(sudo docker run -d -p 5984--volumes-from $COUCH1 shykes/couchdb:2013-05-03)

在第二个数据库中浏览数据

    $ HOST=localhost
    
    $ URL="http://$HOST:$(sudo docker port $COUCH2 5984 | grep -o '[1-9][0-9]*$')/_utils/"
    
    $ echo "Navigate to $URL in your browser. You should see the same data as in the first database"'!'



恭喜，你现在已经运行了两个Couchdb容器，完成了除了数据外的隔离。