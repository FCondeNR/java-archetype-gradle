# Spring Boot アーキタイプ (Gradle, Java 21)

このリポジトリは、Gradle と Java 21 をベースにした Spring Boot プロジェクトのアーキタイプを提供します。
将来的な監視ツール（Prometheus）および CI/CD ツール（Jenkins）との統合を見据えたテンプレートとしてご利用いただけます。

---

## 📋 説明

* **Java 21** を Gradle toolchain で設定
* **Spring Boot 3.x**（`spring-boot-starter-web`、`spring-boot-starter-actuator`）
* REST コントローラーのサンプル（`/hello`）
* Prometheus 互換のメトリクスエンドポイント
* Jenkins パイプライン用の雛形（autobuild & deploy）

---

## 🚀 要件

* **Java 21 JDK**
* **Gradle Wrapper**（プロジェクト同梱）
* **Docker**（Prometheus/Jenkins 実行時に推奨）
* **IDE**（IntelliJ IDEA など）

---

## 🏗️ プロジェクト構成

```text
java-archetype-gradle/
├── build.gradle.kts       # Gradle Kotlin DSL スクリプト
├── settings.gradle.kts    # プロジェクト名設定
├── src/
│   ├── main/
│   │   ├── java/com/fjnunez/javarchetype/
│   │   │   ├── Main.java                   # @SpringBootApplication
│   │   │   └── _rest_/HelloWorldController.java
│   │   └── resources/
│   │       └── application.properties      # プロパティ設定
│   └── test/                              # ユニットテスト
├── .gitignore
└── README.md                              # ドキュメント（このファイル）
```

---

## ⚙️ 設定と実行

1. **リポジトリをクローン**

   ```bash
   git clone https://tu-repositorio.git
   cd java-archetype-gradle
   ```

2. **Gradle Wrapper で起動**

   ```bash
   ./gradlew bootRun
   ```

   > Windows 環境では `gradlew.bat bootRun`

3. **動作確認**

   * ブラウザまたは `curl` で次の URL にアクセス:

     ```
     http://localhost:8080/hello
     ```
   * **Hello World!** が表示されれば成功です。

---

## 📈 メトリクスと Prometheus

Actuator が提供する Prometheus 互換のエンドポイント:

```
http://localhost:8080/actuator/prometheus
```

`prometheus.yml` の例:

```yaml
scrape_configs:
  - job_name: 'java-archetype'
    metrics_path: '/actuator/prometheus'
    static_configs:
      - targets: ['<YOUR_HOST>:8080']
```

Docker で Prometheus を起動する例:

```bash
docker run -d -p 9090:9090 \
  -v $(pwd)/prometheus.yml:/etc/prometheus/prometheus.yml \
  prom/prometheus
```

---

## 🤖 Jenkins (自動ビルド & パイプライン)

### Jenkinsfile のサンプル

```groovy
pipeline {
  agent any

  tools {
    jdk 'jdk21'
    gradle 'gradle-wrapper'
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Build') {
      steps { sh './gradlew clean build' }
    }

    stage('Test') {
      steps { sh './gradlew test' }
      post {
        always { junit '**/build/test-results/**/*.xml' }
      }
    }

    stage('Docker Build') {
      steps {
        sh 'docker build -t java-archetype:latest .'  
      }
    }

    stage('Prometheus Integration') {
      steps {
        echo 'prometheus.yml を確認し、Scrape 設定を行ってください'
      }
    }
  }

  post {
    success {
      echo 'ビルドとテストが成功しました。'
    }
    failure {
      echo 'パイプラインでエラーが発生しました。'
    }
  }
}
```

1. Jenkins で Pipeline ジョブを作成し、リポジトリを指定
2. **Global Tool Configuration** で JDK と Gradle を設定
3. Docker が利用可能なエージェントで実行

---

## 📚 今後の拡張

* Spring プロファイル管理 (`application-dev.yml`, `application-prod.yml`)
* Docker Compose による Prometheus/Grafana/Jenkins Agent のオーケストレーション
* Gradle プラグイン: SpotBugs, SonarQube などの品質ツール
* Prometheus + Grafana アラート設定
* Jenkins X / Argo CD を用いた Kubernetes への継続的デプロイ

---

この README を基に、監視と CI/CD に対応したアーキタイプが完成します。
追加の要望やモジュールが必要な場合はお知らせください。


BY FJNUNEZ