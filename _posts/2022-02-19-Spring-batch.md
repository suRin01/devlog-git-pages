---
layout: post
title: "인텔리제이로 Spring batch 입문하기"
subtitle: ""
date: 2022-02-19 12:30:15 +0900
background: "/img/posts/05.jpg"
---

# 스프링 배치

프로젝트를 진행하면서 데이터베이스에 (내기준) 대용량 일괄 처리가 필요한 부분이 생겼다.

api에서 데이터를 받아오고, 적정한 수준까지 가공한 다음 디비에 주입 및 업데이트 해주는 작업이 필요하게 되었는데,
이 부분에 대해서 배치성 작업에 특화된 프레임워크를 찾아보다가, 프로젝트를 스프링으로 진행하고 있기도 하니 스프링 배치를 사용하자는 결론에 이르게 되었다.

## 스프링 배치 특징

스프링 배치는 스프링 사용자들이 익숙한 설정을 이용해서 편리하게 배치 기능을 설정 할 수 있다.
다양한 라이터와 리더들을 제공하기 때문에 대용량 프로세스를 처리하는데 필요한 로깅, 검증, 청크등을 통해서 비즈니스 로직에 집중할 수 있게 된다.
별도의 스케쥴러를 지원하지는 않지만 쿼츠나 스프링 클라우드 태스크 등을 이용해서 스케쥴러에 연동하여 사용할 수 있게 된다.

![](/img/posts/22_02_22/arch.png)
[출처](https://terasoluna-batch.github.io/guideline/5.0.0.RELEASE/en/Ch02_SpringBatchArchitecture.html)
스프링 배치에서 어떻게 작업이 이루어지는지의 플로우를 다이어그램으로 설명하는 사진이다.

주가 되는 프로세싱 플로우는

1. JobLauncher가 job scheduler에 의해 초기화 되고,
2. job이 jobLauncher에 의해 실행되고,
3. Step이 job에 의해 실행되고,
4. Step이 ItemReader를 통해 데이터를 가져 온 후,
5. Step에서 ItemProcessor를 통해 데이터가 처리되고,
6. Step에서 ItemWriter를 통해 처리된 데이터를 반환한다.

이때 Job과 Step은 1:N 관계이고, Step과 Tasklet은 1:1 관계를 가지고, 실행은 job단위로 이루어진다.

## Intellij 스프링 배치 프로젝트 생성

간단한 튜토리얼 예제를 가지고 simple tasklet 생성, 스케쥴러 연동까지 하나씩 해보도록 하자.

![](/img\posts\22_02_22\K-20220228-194226.png)
우선 스프링 이니셜라이저에서 기본적인 사항들을 입력하고, type은 gradle로 하자.
[gradle과 maven의 차이점](https://hyojun123.github.io/2019/04/18/gradleAndMaven/)

![](/img\posts\22_02_22\K-20220303-115930.png)
추가할 디팬던시는 Lombok, Spring Boot DevTools, Mysql Driver, Spring Batch, Spring Data JPA가 필요하다.

기존에 많이 소개되는 H2 Database를 사용하면 배치에서 사용되는 메타데이터 테이블을 자동으로 생성해준다는 이점이 있지만,
우리는 데이터를 빼내고 가공하고 다시 넣어주는것이 목적이기 때문에 Mysql 드랑리버와 부수적은 것들을 설치해준다.

![](/img\posts\22_02_22\K-20220301-131108.png)
우선 진입정으로 가서 다음과 같은 어노테이션을 추가해 주자.

해당 어도테이션을 배치 동장에 필요한 기본적인 설정을 등록해주고, JobBuilderFactory, StepBuilderFactory들을 주입받을 수 있는 것이 이 어노테이션을 설정하면서 자동으로 빈이 등록되기 때문이다.

![](/img\posts\22_02_22\K-20220301-131252.png)
다음으로 데이터베이스에 메타데이터용 테이블을 생성해주자

shift를 두번 누르면 창이 하나 뜨는데, 거기서 schema 라고 검색 후 자신의 데이터베이스에 맞는 스키마 파일을 가지고 오면 된다.

우리는 mysql을 사용할 것이기 때문에, schema-mysql.sql 파일을 사용하도록 하자.

![](/img\posts\22_02_22\K-20220301-132001.png)
mysql workbench에서 파일 내 스크립트를 복사하고, 실행시켜주면 된다.

![](/img\posts\22_02_22\K-20220301-132029_LI.jpg)
이후 spring batch 프로젝트의 application.properties에서 다음과 같이 데이터베이스 연결에 필요한 사항들을 입력한다.

예시는 다음과 같다.

```

spring.batch.job.enabled=false
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://ip.address.to.db:port/DataBase
spring.datasource.username=usernameHere
spring.datasource.password=passwordHere

```

![](/img\posts\22_02_22\K-20220301-132247.png)
이후 jpa를 사용할 때 사용하기 좋은 플러그인을 하나 추가적으로 설치해주도록 하자.
옵션사항이지만, jpa 엔티티 생성이나 레포지토리 생성때 매우 사용하기 편리하다.
강추

해당 메뉴는 상단바 좌측 file -> setting -> plugins 에서 찾을 수 있다.

![](/img\posts\22_02_22\K-20220301-132356.png)
플러그인 설치 후, 인텔리제이를 재실행하면 우측에 다음과 같은 창에서 데이터베이스를 연결시켜주자.

mysql을 사용하니까 mysql을 클릭을 하고,,

![](/img\posts\22_02_22\K-20220301-134149_LI.jpg)
다음과 같이 필요한 데이터들을 입력시킨 후 테스트 커넥션을 누르면 연결 성공 시 다음과 같이 창이 뜰 것이다.

Apply를 누르고 다음 작업을 진행하자.

![](/img\posts\22_02_22\K-20220301-134200.png)
다음 좌측 jpa structure 탭에서 Entitiy from DB를 눌러, 테스트용으로 만들어놓은 테이블로 부터 엔티티를 생성하자.

![](/img\posts\22_02_22\K-20220301-134207_LI.jpg)
나의 경우 spots이라는 테이블을 미리 생성해 두었고, 이 테이블의 엔티티를 생성할 예정이다.

![](/img\posts\22_02_22\K-20220301-134228_LI.jpg)
이와 유사하게 설정한 후, package에서 entities라는 패키지를 새로 생성할 것이기 때문에 위와 같이 빨간 글씨로 나타나게 된다.

ok를 눌러서 진행하자.

![](/img\posts\22_02_22\K-20220301-134239_LI.jpg)
그러면 이렇게 해당 엔티티를 어떻게 설정할지, 엔티티의 아이템들의 형과 메핑 타입들을 설정하는 창이 나온다.

![](/img\posts\22_02_22\K-20220301-134350_LI.jpg)
생성된 엔티티로 가서, Lombok에 포함된 @Getter 어노테이션으로 getter를 생성해주고, 아래에 있는 getter와 setter들을 지워준다.

![](/img\posts\22_02_22\K-20220301-134437_LI.jpg)
이제 생성된 엔티티로 repository를 생성하자.

![](/img\posts\22_02_22\K-20220301-134455.png)
아까 만든 엔티티를 등록하고, 패키지를 repositories로 설정한 후 생성한다.

![](/img\posts\22_02_22\K-20220301-134515_LI.jpg)
이제 생성된 레포지토리를 가지고 메소드를 추가해보자.
아까 깔았던 jpa buddy에서 지원하는 jpa palette에서 find collection을 드래그 앤 드랍으로 끌어다주면 메소드가 생성된다
짱편함

![](/img\posts\22_02_22\K-20220301-134614.png)
찾아올 attribute에 대해서, 값을 어떤 기준으로 받아올것인지에 대해 operation으로 설정한다.
여기서는 id가 같은 컬럼을 가져오기 위한 조건으로, attribute가 id고 operation이 is인 경우에 찾아오도록 설정했다.

![](/img\posts\22_02_22\K-20220301-134618.png)
그러면 요롷게 딱
메소드가 생성된다.

이제 이 레포지토리를 사용해서 디비에서 엔티티들을 가져와서 콘솔창에 뿌려주는 작업을 해보자!

```java
//TutorialTasklet.java
package com.springtutorial.demo.tasklets;

import com.springtutorial.demo.repositories.SpotRepository;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.batch.core.StepContribution;
import org.springframework.batch.core.scope.context.ChunkContext;
import org.springframework.batch.core.step.tasklet.Tasklet;
import org.springframework.batch.repeat.RepeatStatus;

@RequiredArgsConstructor
public class TutorialTasklet implements Tasklet {

    private final SpotRepository spotRepository;

    @Override
    public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext ) throws Exception{
        System.out.println("executed tasklet");

        System.out.println(spotRepository.findByIdIs(1));

        return RepeatStatus.FINISHED;

    }
}

```

우선 테스크를 만들어 준다.
단순한 로그 한줄 뿌려주고,
spotRepository에서 findByIdIs()메소드를 통해서 데이터를 하나 가져온다.

그 후 테스크가 종료되었다는 RepeatStatus.FINISHED를 리턴해주며 끝난다

이제 이 테스크를 실행시킬 job과 스케쥴로 job을 돌려보자!

```java
package com.springtutorial.demo.jobs;

import com.springtutorial.demo.repositories.SpotRepository;
import com.springtutorial.demo.tasklets.TutorialTasklet;
import lombok.AllArgsConstructor;
import org.springframework.batch.core.Job;
import org.springframework.batch.core.Step;
import org.springframework.batch.core.configuration.annotation.JobBuilderFactory;
import org.springframework.batch.core.configuration.annotation.StepBuilderFactory;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
@AllArgsConstructor
public class TutorialConfig {

    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;
    private final SpotRepository spotRepository;

    @Bean
    public Job tutorialJob(){
        return jobBuilderFactory.get("tutorialJob")
                .start(tutorialStep())
                .build();
    }

    @Bean
    public Step tutorialStep() {
        return stepBuilderFactory.get("tutorialStep")
                .tasklet(new TutorialTasklet(spotRepository)) // Tasklet 설정
                .build();
    }
}

```

일단 잡과 스탭을 생성해준다.
그리고 step에서 사용되는 tasklet에 spotRepository가 사용되기때문에 new TutorialTasklet(spotRepository)로 생성해주었다

```java
package com.springtutorial.demo.schedulers;

import lombok.RequiredArgsConstructor;
import org.springframework.batch.core.Job;
import org.springframework.batch.core.JobExecutionException;
import org.springframework.batch.core.JobParametersBuilder;
import org.springframework.batch.core.launch.JobLauncher;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

import java.time.LocalDateTime;

@Component
@RequiredArgsConstructor
public class TutorialScheduler {

    private final Job job;
    private final JobLauncher jobLauncher;

    // 5초마다 실행
    @Scheduled(fixedDelay = 5 * 1000L)
    public void executeJob () {
        try {
            jobLauncher.run(
                    job,
                    new JobParametersBuilder()
                            .addString("datetime", LocalDateTime.now().toString())
                            .toJobParameters()  // job parameter 설정
            );
        } catch (JobExecutionException ex) {
            System.out.println(ex.getMessage());
            ex.printStackTrace();
        }
    }
}


```

@Scheduled 어노테이션을 통해서 5초마다 실행으로 설정하고, 실행!

![](/img\posts\22_02_22\K-20220301-142215.png)
실행을 하면,
이렇게 5초마다 실행을 한다!

이상으로 가장 기본적인 배치를 짜보았다.

필요한 부분
