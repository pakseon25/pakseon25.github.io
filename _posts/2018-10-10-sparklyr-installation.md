---
layout: post
title: sparklyr 설치 (단일 노드)
categories:
  - R
---

[sparklyr](http://spark.rstudio.com/)은 Apache Spark(이하 Spark) 위에서 동작하는 만큼 여러 서버에서 동작시키는 것이 이상적이겠지만 sparklyr의 여러 기능을 시험할 때는 랩탑이나 개인용 PC 한 대에서 작업하는 것도 그리 나쁜 선택은 아닐 것입니다.

sparklyr은 RStudio에서 만든 것으로 알고 있는데 그 때문인지 RStudio와 궁합이 좋습니다.
이후 작성하는 모든 코드는 RStudio에서 실행한다고 가정하겠습니다.

이 문서를 작성하던 시점에는 sparklyr 최신 버전이 "0.9.1"이 최신 버전이고 이 문서에도 "0.9.1" 설치를 기준으로 작성했습니다.
설치와 튜토리얼은 [sparklyr 공식 홈페이지](http://spark.rstudio.com/)를 참고했습니다.


## sparklyr 설치

사실 sparklyr은 R 패키지로 제공되기 때문에 설치 과정이 아주 쉽습니다.

- CRAN에 올라온 패키지 설치
```r
install.packages("sparklyr")
```

- github에 있는 최신 버전을 설치
```r
devtools::install_github("rstudio/sparklyr")
```

위 명령을 입력하고 나면 sparklyr 패키지가 설치됩니다.


## Apache Spark 설치

sparklyr은 Spark 위에서 동작하기 때문에 Spark 설치가 필요합니다.
Spark를 이미 설치해두신 분이라면 이 부분은 건너뛰셔도 됩니다.
여기서는 단일 노드 실행만을 가정하여 복잡한 설치와 설정 과정은 제외하겠습니다.
sparklyr에서 Spark 설치를 위한 여러 함수를 제공하므로 이 함수를 활용해서 설치하겠습니다.


### sparklyr로 Spark 설치

먼저 sparklyr 패키지를 사용할 수 있도록 패키지를 로드하겠습니다.
그리고 `spark_available_versions()` 함수로 sparklyr에서 설치를 지원하는 Spark 버전을 확인합니다.

```r
library(sparklyr)
spark_available_versions()
```

"1.6.0"부터 "2.3.1"까지 여러 버전이 출력되는 데 이 중에서 "2.3.1"을 설치하겠습니다.

```r
spark_install(version = "2.3.1")
```

이 명령을 실행하면 다음과 같은 화면이 표시됩니다.

```
Installing Spark 2.3.1 for Hadoop 2.7 or later.
Downloading from:
- 'https://archive.apache.org/dist/spark/spark-2.3.1/spark-2.3.1-bin-hadoop2.7.tgz'
Installing to:
- '~/spark/spark-2.3.1-bin-hadoop2.7'
trying URL 'https://archive.apache.org/dist/spark/spark-2.3.1/spark-2.3.1-bin-hadoop2.7.tgz'
Content type 'application/x-gzip' length 225883783 bytes (215.4 MB)
=
```

다운로드 주소가 "https://archive.apache.org/dist/"로 시작하는데 "https://archive.apache.org/dist/"로 들어가보면 "Please do not download from apache.org! If you are currently at apache.org and would like to browse, please visit a nearby mirror site instead."라는 메시지가 눈에 띕니다.
더 상위 페이지로 이동해보면 IP당 하루에 5GB 이상은 다운로드하지 말라고 하니 크게 문제는 없을 것 같지만 제 경우에는 다운로드 속도가 너무 느렸습니다.

혹시 다운로드가 속도가 너무 느려서 안되겠다 하시는 분은 [http://spark.apache.org](http://spark.apache.org)에서 패키지를 직접 다운로드하신 다음 설치하는 것을 추천드립니다.
이 때는 위에서 설치하려던 버전이 없을 수 있는데 [Semantic Version 2.0 문서](https://semver.org/lang/ko/)에 따라 버전이 앞에서 부터 두 자리까지 같다면 테스트 목적으로는 크게 문제가 되지 않을 것이라 생각됩니다.
저는 2.3.1 대신 가장 최근 릴리즈인 2.3.2 버전을 설치했습니다.

압축된 패키지를 설치할 때는`spark_install_tar()` 함수를 사용합니다.
함수에 전달하는 인자는 패키지 파일 경로입니다.

```r
spark_install_tar("~/spark-2.3.2-bin-hadoop2.7.tgz")
```

Spark는 `spark_install_dir()`에서 가리키는 위치에 설치됩니다.
기본 위치는 사용자 홈디렉터리의 spark 디렉터리입니다.

```r
spark_install_dir()
# 출력: /home/pak25/spark
```

설치 후에는 `spark_installed_versions()` 함수로 Spark 버전과 설치 위치를 확인하실 수 있습니다.

```r
spark_installed_versions()
# 출력:
#
#   spark    hadoop  dir
# 1 2.3.2    2.7     /home/pak25/spark/spark-2.3.2-bin-hadoop2.7
```

[sparklyr 문서](http://spark.rstudio.com/#connecting-to-spark)에서는 아래 코드로 Spark에 연결하라고 하지만 저는 이렇게 했을 때 오류가 발생했습니다.

```r
library(sparklyr)
sc <- spark_connect(master = "local")
# 출력:
#
# * Using Spark: 2.3.2
# Error in spark_version_from_home(spark_home, default = spark_version) : 
#   Failed to detect version from SPARK_HOME or SPARK_HOME_VERSION. Try passing the spark version explicitly.
```

오류 메시지를 보면 `SPARK_HOME` 설정이 잘못된 것으로 보입니다.
`spark_installed_version()`로 확인했던 경로를 `spark_connect()`에 spark_home 파라미터로 같이 넘겨줍니다.

```r
library(sparklyr)

sc <- spark_connect(master = "local",
                    spark_home = "/home/pak25/spark/spark-2.3.2-bin-hadoop2.7/")
```


이것으로 설치와 Spark 연결이 끝났습니다.
sparklyr에 대한 간단한 튜토리얼이 필요하시다면 [Using dplyr](http://spark.rstudio.com/#using-dplyr)이 문서에서도 설치 바로 다음에 이어서 등장하는 만큼 간단하고 쉬워서 추천드립니다.

