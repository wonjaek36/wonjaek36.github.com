---
layout: post
title: "Strategy Pattern 활용"
date: 2020-02-06 17:39 +0900
category: [development, design_pattern]
tags: [design_pattern]
---

## Strategy Pattern 특징
&nbsp;
* 프로그램 로직에서 자주 변경될 수 있는 부분을 동적으로 수정할 수 있게 캡슐화함
* 캡슐화된 인터페이스를 필요에 맞게 구현하여 행위를 유연하게 변경
* 행위 패턴 중의 하나
* JDK 1.7까지는 2가지 방식, 새로운 클래스 구현 또는 익명클래스 구현으로 전략패턴을 활용
* JDK 1.8부터 람다 식(Lambda expression)을 활용하여 전략패턴을 활용할 수 있습니다.

&nbsp;  

## Strategy Pattern 활용
&nbsp;
Strategy 패턴의 가장 대표적인 예로, 클라이언트들이 자원에 접근하여 다양한 프로세스를 처리할 때 입니다. 이러한 문제는 Execute around method pattern 이라고 하기도 합니다.  

먼저, Client A와 B가 공통의 파일 file.txt에 접근하는데, A와 B는 각각 홀수 그리고 짝 수번째 줄을 따로 출력한다고 가정하겠습니다.  

여기서 A와 B의 공통점은 file.txt을 열고, 읽고, 다 작업한 후에 file.txt를 close하는 것입니다. 그리고, A는 데이터를 홀수 줄이면 출력, 짝수 줄이면 넘어가는 것입니다. (B는 그 반대)  

가장 단순한 방식으로는 아래와 같이 두 가지 클라이언트를 직접 구현하면 됩니다. (ClientB는 process의 if문만 살짝 바꾸면됩니다.)  

{% highlight java %}
import java.io.*;

public class ClientA {

    String filename;
    BufferedReader br;

    public ClientA(String filename) {

        this.filename = filename;
    }

    public void openFile() throws FileNotFoundException {

        br = new BufferedReader(new FileReader(new File(filename)));
    }

    public void process() throws IOException {

        String line;
        for(int i=1;(line=br.readLine())!=null;i++) {

            if(i % 2 == 1) {
                System.out.println(line);
            }
        }
    }

    public static void main(String[] args) {

        String filename = "file.txt";
        ClientA clientA = new ClientA(filename);
        try {
            clientA.openFile();
            clientA.process();
        } catch(Exception e) {
            e.printStackTrace();
        }
    }
}
{% endhighlight %}

하지만, 이러한 방식은 그다지 좋지 못 합니다. 프로그래밍의 DRY(Don't Repeat Yourself) 원칙을 지키지 못 할 뿐더러, 새로운 클라이언트C가 다른 요구사항을 알려주면 처음부터 다시 작성해야되기 때문입니다. 예제코드야 약 40줄이고, 간단하기 때문에 구현하는데 문제가 별로 없지만 큰 프로젝트에서는 이러한 방식으로 구현했다가는 퇴근을 못 하겠죠.

### 추상클래스와 인터페이스
&nbsp;  
DRY 원칙을 지키기 위해, 추상클래스 Client.java와 인터페이스 FileProcessor.java를 추가했습니다.

**Client.java**

{% highlight java %}
import java.io.BufferedReader;
import java.io.File;
import java.io.FileNotFoundException;
import java.io.FileReader;

public abstract class Client {

    String filename;
    BufferedReader br;

    public Client(String filename) {
        this.filename = filename;
    }

    public void openFile() throws FileNotFoundException {
        br = new BufferedReader(new FileReader(new File(filename)));
    }
}
{% endhighlight %}


**FileProcessor.java**
{% highlight java %}
import java.io.*;
public interface FileProcessor {

    void process() throws IOException;
}
{% endhighlight %}

**ClientA.java**
{% highlight java %}
import java.io.*;

public class ClientA extends Client implements FileProcessor {

    public ClientA(String filename) {
        super(filename);
    }

    @Override
    public void process() throws IOException {

        String line;
        for(int i=1;(line=br.readLine())!=null;i++) {

            if(i % 2 == 1) {
                System.out.println(line);
            }
        }
    }

    public static void main(String[] args) {

        String filename = "file.txt";
        ClientA clientA = new ClientA(filename);
        try {
            clientA.openFile();
            clientA.process();
        } catch(Exception e) {
            e.printStackTrace();
        }
    }
}
{% endhighlight %}

하지만, 아직도 DRY 원칙을 어기는 부분이 있습니다. 바로 process 내부 for문입니다. 파일을 읽어드리는 for문은 모든 Client가 동일하기 때문에 Client들을 추가할때마다 중복될 것이 확실합니다. Client별로 다른 행위를 하게 되는 부분은 읽어드린 파일의 한 줄을 기준에 따라 출력할지 말지 결정하는 것 입니다.  

### 인터페이스 개선
&nbsp;  
중복되는 부분을 더 제거하기 위해서 interface FileProcessor를 interface LineClassifier로 재정의하고, 새로운 함수 accept를 정의합니다. 이 accpet함수는 line의 index를 받아서, 해당 라인을 출력할 것인지 아니면, 출력하지 않고 넘어갈지 결정할 수 있습니다. 이러한 방식으로 interface를 구현하면, 어떤 라인들을 출력할지 결정하는 것은 ClientA와 ClientB를 구현할 때 결정할 수 있습니다.

변경된 interface에 맞게 추상클래스 Client, 클래스 ClientA와 ClientB를 다음과 같이 구현합니다. (여기서 구현의 편의상 interface를 ClientA와 ClientB 내부 클래스로 구현하였지만, 일반적으로는 독립적인 클래스로 구현합니다.)

**LineClassifier.java**
{% highlight java %}
import java.io.*;
public interface LineClassifier {

    boolean accept(int index);
}
{% endhighlight %}

**Client.java**
{% highlight java %}
import java.io.*;

public abstract class Client {

    String filename;
    BufferedReader br;

    public Client(String filename) {
        this.filename = filename;
    }

    public void openFile() throws FileNotFoundException {
        br = new BufferedReader(new FileReader(new File(filename)));
    }

    public void process(LineClassifier lineClassifier) throws IOException {

        String line;
        for(int i=1;(line=br.readLine())!=null;i++) {

            if(lineClassifier.accept(i)) {
                System.out.println(line);
            }
        }
    }
}
{% endhighlight %}


**ClientA.java**
{% highlight java %}
public class ClientA extends Client {

    public ClientA(String filename) {
        super(filename);
    }

    public static class ClientALineClassifier implements LineClassifier {

        @Override
        public boolean accept(int index) {
            return index % 2 == 1;
        }
    }

    public static void main(String[] args) {

        String filename = "file.txt";
        ClientA clientA = new ClientA(filename);
        try {
            clientA.openFile();
            clientA.process(new ClientALineClassifier());
        } catch(Exception e) {
            e.printStackTrace();
        }
    }
}
{% endhighlight %}

### 익명클래스 활용
&nbsp;  
이제 Client는 매우 편하게 확장할 수 있게 되었습니다. 새로운 클라이언트가 나타나서 3의 배수(3, 6, 9, 12 ... ) 줄 번호 라인들만 출력해달라는 요청을 하더라도, Client를 처음부터 구현하는 것이 아니라 LineClassifier만 구현하면 빠르게 사용할 수 있습니다.

그렇지만, ClientA와 ClientB처럼 interface를 클래스로 구현하는 것이 아니라 간단하게 익명클래스를 이용하여 더 간단하게 할 수 있는 방법이 있습니다.

**ClientC.java**
{% highlight java %}
public class ClientC extends Client {

    public ClientC(String filename) {
        super(filename);
    }

    public static void main(String[] args) {

        String filename = "file.txt";
        ClientC clientC = new ClientC(filename);
        try {
            clientC.openFile();
            clientC.process(new LineClassifier() {

                @Override
                public boolean accept(int index) {
                    return index % 3 == 0;
                }
            });

        } catch(Exception e) {
            e.printStackTrace();
        }
    }
}
{% endhighlight %}

LineClassifer를 구현하는 것은 단순하기 때문에, ClientC와 같이 익명클래스를 정의해서 구현하는 것이 좋습니다. 이와 같은 방식을 이용하면, 마치 함수에 객체를 전달하는 것이 아닌 코드를 전달하는 것과 같은 착각을 불러올만큼 코드 구현을 줄일 수 있습니다.

### Lambda식
&nbsp;  
하지만, 여기서 조금 더 간단한 방법이 있습니다. JDK 1.8로 들어오면서, Java는 함수형 언어 기능들을 채용하기 시작했습니다. 그 중에 하나가 lambda 식입니다. 익명 클래스는 기존의 인터페이스를 직접 구현하는 것보다 코드를 줄여줬지만, 아직까지도 함수에 객체를 직접 개발자가 전달하고 있습니다. 하지만, 람다(Lambda) 식을 이용하면 함수의 내용만 직접 구현하고, 나머지 부분은 언어에게 맡길 수 있습니다.

**ClientC.java**
{% highlight java %}
public class ClientC extends Client {

    public ClientC(String filename) {
        super(filename);
    }

    public static void main(String[] args) {

        String filename = "file.txt";
        ClientC clientC = new ClientC(filename);
        try {
            clientC.openFile();
            clientC.process(index -> (index % 3 == 0));

        } catch(Exception e) {
            e.printStackTrace();
        }
    }
}
{% endhighlight %}

람다식을 이용하면 clientC.process 호출부분이 한 줄로 매우 간단해진 것을 확인할 수 있습니다. 특히, 람다식은 캡슐화된 인터페이스를 객체로 직접 구현하는 부분을 간략화하여, 실제 라인을 구분하는데 필요한 부분만 구현할 수 있도록 도와줍니다. 람다식을 활용하려면, 인터페이스가 함수형 인터페이스 제약사항이 있지만, 잘 활용하면 위와 같이 코드를 매우 깔끔하게 짤 수 있습니다. 특히, strategy 패턴처럼 일부분의 코드만 자주 변경해야되는 경우에 매우 잘 활용할 수 있습니다.

참고자료:
* 함수형 인터페이스 - [Annotation FunctionalInterface](https://docs.oracle.com/javase/8/docs/api/java/lang/FunctionalInterface.html), [java.util.function](https://docs.oracle.com/javase/8/docs/api/java/util/function/package-summary.html)
* [Strategy Pattern Wiki](https://en.wikipedia.org/wiki/Strategy_pattern)
* [스트래티지 패턴이란](https://gmlwjd9405.github.io/2018/07/06/strategy-pattern.html)
* [Modern Java in Action](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9791162242025&orderClick=LAG&Kc=)
