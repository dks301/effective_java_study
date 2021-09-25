# 2장. 객체 생성과 파괴

> GOAL  
> 1. 객체를 만들어야 할 때와 만들지 말아야 할 때를 구분하기  
> 2. 올바른 객체 생성 방법과 불필요한 생성을 피하는 방법  
> 3. 제때 파괴됨을 보장하고, 파괴 전에 수행해야 할 정리 작업을 관리하기

# 아이템9. try-finally보다는 try-with-resources를 사용하라

자바 라이브러리에는 close메서드를 호출해 직접 닫아줘야 하는 자원이 많다.  
ex) `IuputStream, OutputStream, java.sql,Connection`  
자원 닫기는 클라이언트가 놓치기 쉬워서 성능 문제로도 이어지기도하는데, 이런 자원 중 상당수가 안전망으로 finalizer를 활용하고는 있지만, finalizer는 믿을만하지 못하다.

전통적으로는 자원이 제대로 닫힘을 보장하는 수단으로 `try-finally`가 쓰였다.  
자원이 둘 이상이라면 `try-finally` 방식은 너무 지저분하다. `close()`작성도 너무 난해하다.

```java
public static String firstLineOfFile(String path) throw IOException {
    BufferedReader br = new BufferedReader(new FileReader(path));
    try {
        return br.readLine();
    } finally {
        br.close();
    }
}

static void copy(String src, String dst) throws IOException {
    InputStream in = new FileInputStream(src);
    try {
        OutputStream out = new FileOutputStream(dst);
        try {
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while ((n = in.read(buf)) >= 0)
            out.write(buf, 0, n);
        } finally {
            out.close();
        }
    } finally {
        in.close();
    }
}
```

## Try-with-Resource를 사용하라

짧고 읽기 수월할 뿐 아니라, 문제를 진단하기도 훨씬 좋다.

```java
public static String firstLineOfFile(String path) throw IOException {
    try (BufferedReader br = new BufferedReader(new FileReader(path))) {
        return br.readLine();
    } catch (Exception e) {
        return defaultVal;
    }
}

static void copy(String src, String dst) throws IOException {
    try (InputStream in = new FileInputStream(src);
        OutputStream out = new FileOutputStream(dst)) {
        byte[] buf = new byte[BUFFER_SIZE];
        int n;
        while ((n = in.read(buf)) >= 0)
        out.write(buf, 0, n);
    }
}
```

## 결론

꼭 회수해야하는 자원을 다룰 때는 `Try-with-Resource`를 사용하자.  
만들어지는 예외정보도 훨씬 유용하다. 정확하고 쉽게 자원회수가 가능하다.

## 참고

자료는 이펙티브 자바 책과 백기선님의 강의를 들으며 작성했다.  
[\[이펙티브자바 - 아이템9\] try-finally보다는 try-with-resources를 사용하라](https://youtu.be/zqjZBSqHs0s)