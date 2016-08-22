---
layout: post
title:  "죽지 않는 서비스 만들기"
date:   2016-08-22
---
앱 락커 기능을 개발하던 도중 죽지 않는 서비스가 필요해서 관련 자료를 조사하였습니다.<br />
죽지 않는 서비스의 핵심은 startForeground로 서비스를 등록하는 것입니다.<br />
근데 여기서 문제가 하나 발생하였습니다.<br />
아이스크림 샌드위치 버전 이상부터는 startForeground로 앱을 등록하게 되면 Notification이 보이게 됩니다.<br />
이부분은 그냥 넘어가도 되겠지만 요구 사항중에 Notification에 앱이 등록된게 보여지면 안되기 때문에<br />
Notification을 제거 하는 방법을 사용하였습니다.<br />
<br />
그리고 주의 해야할 사항이 있습니다.<br />
<br />
1.Notification 객체.setProperty(Notification.PRIORITY_MIN)

2.단말기마다 등록하고 해제하는 부분의 시간차가 있으므로 딜레이 시간을 적용하였습니다.

이걸 해주지 않으면 동작 되지 않을수도 있습니다.<br />
[관련 블로그 ](http://iw90.tistory.com/155)
<br />


{% highlight ruby %}

public int onStartCommand(Intent intent, int flags, int startId) {
  startForeground(1,new Notification());
        Observable.timer(20, TimeUnit.MILLISECONDS)
                .subscribeOn(Schedulers.computation())
                .subscribe(new Action1<Long>() {
                    @Override
                    public void call(Long aLong) {
                        NotificationManager nm = (NotificationManager) getSystemService(NOTIFICATION_SERVICE);
                        Notification notification;
                        notification = new Notification.Builder(getApplicationContext())
                                .setContentTitle("")
                                .setContentText("")
                                .setPriority(Notification.PRIORITY_MIN)
                                .build();
                        nm.notify(startId, notification);
                        nm.cancel(startId);
                    }
                }, new Action1<Throwable>() {
                    @Override
                    public void call(Throwable throwable) {

                    }
                });
  return Service.START_STICKY;
}

{% endhighlight %}

핵심 코드 입니다.<br />
startForeground 를 실행하고 NotificationManager 를 통해서 notification 을 등록 합니다.<br />
notification 을 startForeground 시 등록했던 아이디와 동일하게 등록후 cancel 을 하면  startForeground 를 사용하면서<br />
notification 을 보여주지 않을 수 있습니다.
