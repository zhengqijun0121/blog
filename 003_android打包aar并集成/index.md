# Android 打包 aar


# Android 打包 aar 并集成

## Android 打包 aar

1. 运行编译命令

```shell
./gradlew build
```

## Gradle 本地集成 aar

1. 将生成的 aar 放到 `app/libs` 目录下

2. 修改 `app/build.gradle` 文件

```
android {
    ...
    repositories {
        flatDir {
            dirs 'libs'
        }
    }
}

dependencies {
    ...
    implementation(name:'aar_name', ext:'aar')
}
```

3. 重新编译 app

## Gradle 迁移 Maven 构建

1. 修改 `app/build.gradle` 文件

```
android {
    ...
    repositories {
        maven {
            url 'https://nexus2.com/repository/maven/'
        }
    }
}

dependencies {
    ...
    implementation 'com.demo:demolib:1.0.0-20220211.010237-47@aar'
}
```

2. 重新编译 app


