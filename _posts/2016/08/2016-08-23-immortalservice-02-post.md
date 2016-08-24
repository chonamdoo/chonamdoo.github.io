---
layout: post
title:  "죽지 않는 서비스 만들기(2)"
date:   2016-08-23
---
죽지 않는 서비스 만들기(1)을 이용해서 테스트 한 결과 동작이 잘되는 폰도 있지만
갤럭시 시리즈는 화면을 껐다 켜면 노티가 보이는 문제가 발생하였습니다. <br />
다른 앱들을 디컴파일 해본 결과 특이한 방법을 발견하였습니다.<br />


{% highlight ruby %}

public class ForeGroundService extends Service{
    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        return null;
    }

    @Override
    public void onCreate() {
        super.onCreate();
        startForeground(this);
        stopSelf();
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        stopForeground(true);
    }
    public static void startForeground(Service service){
        if(service != null){
            try{
                Notification notification = getNotification(service);
                if(notification != null){
                    service.startForeground(1220,notification);
                }
            }catch (Exception e){

            }
        }
    }
    public static Notification getNotification(Context paramContext){
        int smallIcon = R.mipmap.ic_launcher;
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
            smallIcon = R.mipmap.l_launcher;
        }
        Notification notification =  new NotificationCompat.Builder(paramContext)
                .setSmallIcon(smallIcon)
                .setPriority(NotificationCompat.PRIORITY_MIN)
                .setAutoCancel(true)
                .setWhen(0)
                .setTicker("").build();
        notification.flags = 16;
        return  notification;
    }
}

{% endhighlight %}

-매니페스트

{% highlight ruby %}
<service android:exported="false" android:name=".service.EcmoService" android:process=":locker" />

{% endhighlight %}

foreground service를 하고싶은 Service에서

{% highlight ruby %}
public void onCreate() {
  super.onCreate();
  ForeGroundService.startForeground(this);
  Intent localIntent = new Intent(this, ForeGroundService.class);
  startService(localIntent);
}

{% endhighlight %}

EcmoService 서비스 사용전<br />
com.couchgram.privacycall/u0a10196 (bg-services)

EcmoService 서비스 사용후<br />
com.couchgram.privacycall/u0a10196 (fg-service)<br />

변경된것을 보실수 있습니다.

명령어는 adb shell dumpsys activity p 하시면 현재 단말기에 설치된 앱들의 상태를 보실수있습니다.
