---
layout: post
title:  "Sqlite 멀티 스레드 접근 관련 삽질 이야기"
date:   2016-08-15
---
기본적으로 sqlite는 하나의 연결만 허용하며 write 중일때는 read를 하지 못하고 read 중일때 write 하지 못한다
보통의 경우에는 문제 되지않지만 현재 내가 개발중인 앱에서는 멀티 쓰레드로 접근을 하는 경우가 발생하여서 문제가 발생하였다
그래서 생각해낸 방법이 write를 할때는 기존 테이블에 하고 read 할때는 가상의 테이블을 이용해서 가져오기로 하였다

가상의 테이블을 생성하는 방법으로 VIEW TABLE 과 VIRTUAL TABLE 두가지를 고려했는데 VIRTUAL TABLE 로 생성하게 되면
사진에서 보듯이 테이블이 너무 많이 생성되어 버리는 문제가 있다

<img src="{{ '/images/2016/08/2016-08-15-sqlite-post/view_table_image.png' | prepend: site.baseurl }}" alt="">

![chrome-web-app-example](/images/2016/08/2016-08-15-sqlite-post/view_table_image.png)


그래서 VIEW TABLE 로 생성을 하기로 정하고 관련 문법을 찾아 봤다


{% highlight ruby %}

CREATE VIEW " + VIEW_TABLE + " AS " +
                " select "+
                COLUMN_BLACK_LIST_NORMALIZED_PHONE+" , "+
                COLUMN_BLACK_LIST_PHONE+" , "+
                COLUMN_BLACK_LIST_PHONE_TOKEN+" , "+
                COLUMN_BLACK_LIST_FILTER_TYPE+" from "+
                DATABASE_TABLE

{% endhighlight %}

대상이 되는 테이블의 컬럼을 가져와서 CREATE VIEW 만 해주면 자동으로 VIEW TABLE 이 생성되고 원본 데이블의 데이터를 알아서 가져온다
