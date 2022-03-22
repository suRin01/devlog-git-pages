---
layout: post
title: "Spring batch에서 Api가 가끔 다운될 때 예외 처리 및 대응"
subtitle: ""
date: 2022-03-22 12:30:15 +0900
background: "/img/posts/05.jpg"
---

## 문제 상황

현재 진행중인 프로젝트에서 한국관광공사 제공 관광지 api와, 기상청 제공 단기 예보 api를 사용하고 있다.

그런데 관광지api는 문제 없이 잘 돌아가는데, 기상청 단기 예보 api는 간혈적으로 다운되거나, 데이터를 받아오는데 상당항 시간(30초 이상)이 걸릴떄가 있다.

이러한 경우에 대처하기 위해서 스프링 배치에 사용된 설정과, 이것저것 방법들에 대해서 설명하고자 한다.

### Retrofit2 클라이언트 설정

retorfit은 http request를 담당하는 라이브러리로, 여기서 커넥션 타임을 설정하는 것으로 응답 반환 시간에 대해서 설정 할 수 있다.
api에 문제가 있는 경우가 아니더라도 네트워크 환경이 좋지 않은 경우에도 활용한다면 좋은 결과를 얻을 수 있다.

```java

import java.util.concurrent.TimeUnit;

public class WeatherClient {
    private static final String BASE_URL = "http://apis.data.go.kr/";
    public static WeatherApi getApiService(){
        return getInstance().create(WeatherApi.class);
    }

    public static Retrofit getInstance(){

        Gson gson = new GsonBuilder()
                .setLenient()
                .create();

        return new Retrofit.Builder()
                .baseUrl(BASE_URL)
                .addConverterFactory(GsonConverterFactory.create(gson))
                .build();
    }

```

이와 같던 기존의 코드에서,

```java

import java.util.concurrent.TimeUnit;

public class WeatherClient {
    private static final String BASE_URL = "http://apis.data.go.kr/";
    public static WeatherApi getApiService(){
        return getInstance().create(WeatherApi.class);
    }

    public static Retrofit getInstance(){
        OkHttpClient okHttpClient = new OkHttpClient.Builder()
                .connectTimeout(1, TimeUnit.MINUTES) // 기본 30초, 1분으로 변경
                .readTimeout(30, TimeUnit.SECONDS)
                .writeTimeout(15, TimeUnit.SECONDS)
                .build();

        Gson gson = new GsonBuilder()
                .setLenient()
                .create();

        return new Retrofit.Builder()
                .baseUrl(BASE_URL)
                .client(okHttpClient)
                .addConverterFactory(GsonConverterFactory.create(gson))
                .build();
    }

```

다음과 같이 okHttpClient 객체를 생성하고 retrofilt 빌더 client로 지정해 준다.
retrofit이 okHttp 라이브러리를 사용한 라이브러리기 때문에, 위와 같은 형태로 사용되게 된다.

retrofit에서 어떠한 옵션 설정이 되어 있지 않을 경우, 커넥션 타임아웃, 읽기 타임아웃, 쓰기 타임아웃은 모두 30초로 설정되어 있다.
이 부분에서 커넥션 타임아웃 부분을 1분으로 설정해주자.

### 서버 다운시 처리

우선 스프링 배치 프로젝트에 있는 스탭 외부에서 데이터를 저장하고 불러올 수 있는 방법이 필요하다.
이 경우에 StepExecution의 ExecutionContext에 key-value로 데이터를 저장하고 불러올 수 있는 방법이 있다.
하지만 이러한 방식으로 데이터를 공유할 경우, 스프링 배치에서는 batch schema에 메타데이터 정보를 넘겨주게 된다.
따라서 공유하고자 하는 데이터의 크기가 클 경우, 과도한 IO 혹은 데이터 사이즈를 초과하는 경우 오류가 발생할 수 있게 된다.

이러한 문제를 해결하기 위해서, 다음과 같은 방법으로 처리하기로 했다.

#### 싱글톤 빈을 이용한 데이터 공유

우선 @Component를 사용해 싱글톤 빈을 하나 생성하고, 멤버 변후로 map을 두어 데이터를 초기화 한다.

```java
package com.surin.apibatchgetter.utilities;

import org.springframework.stereotype.Component;

import java.util.HashMap;
import java.util.Map;

@Component
public class DataShareBean <T> {

    private Map<String, T> shareDataMap;


    public DataShareBean () {
        this.shareDataMap = new HashMap<>();
    }

    public void putData(String key, T data) {
        if (shareDataMap ==  null) {
            System.out.println("Map is not initialize");
            return;
        }

        shareDataMap.put(key, data);
    }

    public T getData (String key){

        if (shareDataMap == null) {
            return null;
        }

        return shareDataMap.get(key);
    }

    public int getSize () {
        if (this.shareDataMap == null) {
            System.out.println(("Map is not initialize"));
            return 0;
        }

        return shareDataMap.size();
    }

    public void updateData(String key, T data){
        if (shareDataMap ==  null) {
            System.out.println("Map is not initialize");
            return;
        }
        shareDataMap.replace(key, data);
    }
}

```

이제 데이터가 필요할 경우, 해당 빈을 주입해서 사용하면 된다!

#### 예외 처리 및 에러시 해당 스탭 재실행

```java

@Slf4j
@Component
@RequiredArgsConstructor
public class WeatherDataCrawler implements Tasklet {

    //Data Share Bean
    private final DataShareBean<List<List<String>>> lambertConicSpotDataShareBean;
    private final WeatherRepository weatherRepository;

    @Override
    public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext ) throws Exception{
        //get working location
        List<List<String>> lambertConicSpots = lambertConicSpotDataShareBean.getData("spots");

        if(lambertConicSpots.size() == 0){
            return RepeatStatus.FINISHED;
        }

        List<String> currentLambertConicSpot = lambertConicSpots.get(0);

        //make request
        log.info("Get Spot List From Api");
        Call<WeatherResponseStructure<Weather>> weatherRequest
                = WeatherClient.getApiService().getWeatherOnXY(PropertyUtil.getProperty("WeatherApi.SecretKey"),
                10000,
                1,
                "JSON",
               (new SimpleDateFormat("yyyyMMdd")).format(new Date()),
                "0500",
                Integer.parseInt(currentLambertConicSpot.get(0)),
                Integer.parseInt(currentLambertConicSpot.get(1)));
        try {
            //call api via nx, ny
            log.info("Execute Weather Api Request");
            WeatherResponseStructure<Weather> response = weatherRequest.execute().body();

            log.info("Got Data From Area Code: {}, {}", currentLambertConicSpot.get(0), currentLambertConicSpot.get(1));
            List<Weather> weatherList = response.getItems();

            log.info("Got Data Count : {}", weatherList.size());
            log.info("Save Crawled Data to Database");
            weatherRepository.saveAll(weatherList);

            log.info("Update Spot Cursor");
            lambertConicSpots.remove(0);
            lambertConicSpotDataShareBean.updateData("spots", lambertConicSpots);
            log.info("Left task count : {}", lambertConicSpots.size());

            return RepeatStatus.CONTINUABLE;
        }catch (IOException e){
            log.info("Error occurred during executing request. do this step again in 30 minutes");
            Thread.sleep(1000 * 60 * 30);
            return RepeatStatus.CONTINUABLE;
        }
    }
}

```

변경된 코드는 다음과 같다.

```java

List<String> currentLambertConicSpot = lambertConicSpots.remove(0);
// ->
List<String> currentLambertConicSpot = lambertConicSpots.get(0);

//-------------------------------------------//
...
log.info("Got Data Count : {}", weatherList.size());
log.info("Save Crawled Data to Database");
weatherRepository.saveAll(weatherList);

log.info("Update Spot Cursor");
lambertConicSpotDataShareBean.updateData("spots", lambertConicSpots);
log.info("Left task count : {}", lambertConicSpots.size());
...
// ->
...
log.info("Got Data Count : {}", weatherList.size());
log.info("Save Crawled Data to Database");
weatherRepository.saveAll(weatherList);

log.info("Update Spot Cursor");
lambertConicSpots.remove(0);
lambertConicSpotDataShareBean.updateData("spots", lambertConicSpots);
log.info("Left task count : {}", lambertConicSpots.size());
...

step 시작 시 data share bean에서 데이터를 받아오고, 작업할 좌표를 remove로 받아오는 대신에
모든 api 처리와 데이터 DB저장까지 끝난 후에 remove를 사용해서 작업할 영억을 제거하고, 만약 작업 도중에 에러가 발생했다면
경고문을 띄워준 후, 30분간 슬립 후 해당 스탭을 RepeatStatus.CONTINUABLE로 재실행하게 된다.


이와 같은 해결책들로 현재 일 1600건 정도의 요청 중에 10~20회 정도의 에러가 발생하지만 문제 없이 처리되고 있다


이상 끝!
```
