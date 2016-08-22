layout: post
title:  "죽지 않는 서비스 만들기"
date:   2016-08-22

앱 락커 기능을 개발하던 도중 죽지 않는 서비스가 필요해서 관련 자료를 조사하였습니다.
죽지 않는 서비스의 핵심은 startForeground로 서비스를 등록하는 것입니다.
근데 여기서 문제가 하나 발생하였습니다.
아이스크림 샌드위치 버전 이상부터는 startForeground로 앱을 등록하게 되면 Notification이 보이게 됩니다.
이부분은 그냥 넘어가도 되겠지만 요구 사항중에 Notification에 앱이 등록된게 보여지면 안되기 때문에
Notification을 제거 하는 방법을 사용하였습니다

[관련 블로그 ](http://iw90.tistory.com/155)

{% highlight ruby %}
@TargetApi(Build.VERSION_CODES.JELLY_BEAN)
    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {

        startForeground(1,new Notification());

        NotificationManager nm = (NotificationManager)getSystemService(NOTIFICATION_SERVICE);
        Notification notification;

        if(Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB){

            notification = new Notification.Builder(getApplicationContext())
                    .setContentTitle("")
                    .setContentText("")
                    .build();
 
        }else{
            notification = new Notification(0, "", System.currentTimeMillis());
            notification.setLatestEventInfo(getApplicationContext(), "", "", null);
        }

        nm.notify(startId, notification);
        nm.cancel(startId);

        return super.onStartCommand(intent, flags, startId);
    }
{% endhighlight %}

핵심 코드 입니다.
  startForeground 를 실행하고 NotificationManager 를 통해서 notification 을 등록 합니다.
  notification 을 startForeground 시 등록했던 아이디와 동일하게 등록후 cancel 을 하면  startForeground 를 사용하면서
  notification 을 보여주지 않을 수 있습니다.
