# RxAndroid Basic: Part1

본 내용은 RxJava(Android)에 대해 기본적인 내용을 이해하고 숙지하기 위해  아래 포스팅의 정리한 것 입니다 . 

[RxAndroid Basic: Part1](https://medium.com/@kurtisnusbaum/rxandroid-basics-part-1-c0d5edcf6850#.gjyh9ew7j)

[샘플 프로젝트](https://github.com/klnusbaum/rxandroidexamples)



## 들어가기 전에..

ReactiveX(Reactive Extention) 또는 Rx는 observable pattern을 이용해 비동기 처리와 이벤트 기반 프로그램을 구성할 수 있도록 해주는 라이브러리 또는 API로 볼 수 있다.

> An API for asynchronous programming with observable streams

> ReactiveX is a combination of the best ideas from the Observer pattern, the Iterator pattern, and functional programming



**RxJava (Reactive Extensions for the JVM)**

- Java VM implementation of Reactive Extensions

- Netflix가 Rx를 Java환경에 적용함

- [RxJava GitHub](https://github.com/ReactiveX/RxJava)

  ​

**RxAndroid (Reactive Extensions for Android)**

- Android applications에서 추가적인 수고로움 없이 RxJava를 사용할 수 있도록 최소한의 class만 추가한 라이브러리
- Main thread나 다른 주어진 Looper에서 로직이 수행될 수 있도록 Scheduler를 제공함
- [RxAndroid GitHub](https://github.com/ReactiveX/RxAndroid)



### Binaries

```
compile 'io.reactivex:rxandroid:1.2.1'
// Because RxAndroid releases are few and far between, it is recommended you also
// explicitly depend on RxJava's latest version for bug fixes and new features.
compile 'io.reactivex:rxjava:1.1.6'
```

- Binary Size - 300KB

  ​

### LICENSE

Apache License, Versoin 2.0



## In Part1

Part1에서는 RxJava를 이용해 비동기로 데이터를 로드하는 과정을 설명합니다. 

안드로이드 프로그래밍을 하다보면, network request를 수행하는 일이 빈번한데 이런 network request의 경우 서버로부터 결과를 받을 때까지 delay가 발생합니다. 따라서, UI가 blocking되지 않도록 해당 작업이 background thread에서 실행되도록 thread를 생성해서 처리합니다. 하지만, 처리 결과를 callback으로 받아서 UI에 반영하기 위해서는 UI Thread로 다시 전달하기 위해 handler 등의 방법을 사용해서 구현하고는 하는데, 코드 간결성이 떨어질 뿐만 아니라 생산성이나 가독성도 좋지 못합니다. RxJava를 이용하면 이와 같은 비동기 데이터를 로드하여 UI에 반영하는 작업을 간결하고 손쉽게 구현이 가능합니다. 



### 기본 컨셉

RxJava에서 가장 핵심적인 두 가지 요소가 Observable과 Observer입니다. 

Observable은 value 를 발행하고, Observer는 반대로 Observable을 구독해서 지켜보다가 value를 받습니다. 



Observer는 다음 세 가지 경우에 대해 추가적인 action들을 취할 수 있습니다. 그리고, 각각의 action들은 Observer의 인터페이스로 추상화되어 있어서 해당 인터페이스를 이용하여 구현할 수 있습니다. 

- Observable이 value를 발행했을 때 -> onNext()
- Observable이 error가 발생했다고 알려줄 때 -> onError()
- Observable이 더 이상 발행할 value가 없다고 알려줄 때 -> onCompleted()



## Example1: The Basics

가장 간단한 예제로 색상 리스트를 보여주는 Activity를 만들어 봅시다. 

여기서 Observable은 Observer가 구독하면 바로 색상 리스트를 value로 발행하고, complete되면 되기 때문에 아주 간단합니다. 이러한 Observable은 Observable.just() 메서드를 이용하여 생성할 수 있는데, 이렇게 생성한 Observable은 아래와 같이 동작합니다. 

1. Observer가 구독 
2. Observable이 바로 value를 발행 
3. Observer의 onNext() 메서드가 Observable.just()에 제공된 매개변수와 함께 호출됨
4. Observer의 onComplete() 콜백이 호출됨

아래는 지금까지 설명한 내용에 대한 간단한 샘플입니다. Observable.just() 메서드에 매개변수로 제공된 getColorList는 non-blocking method라고 생각하고 보시면 이해가 쉽습니다. 

```java
Observable<List<String>> listObservable = Observable.just(getColorList());

listObservable.subscribe(new Observer<List<String>>() { 

    @Override 
    public void onCompleted() { } 

    @Override 
    public void onError(Throwable e) { } 

    @Override
    public void onNext(List<String> colors) {
        mSimpleStringAdapter.setStrings(colors);
    }
});
```

결국 Observable들은 Observer들이 자신들을 구독할 때 어떤 행위(behavior)를 하는냐로 정의된다고 볼 수 있습니다. 

> Observables are primarily defined by their behavior upon subscription.

## Example 2: Asynchronous Loading

이제 우리가 애당초 궁금해했던 비동기로 데이터를 로딩하는 usecase에 대한 예제와 RxJava로 어떻게 구현하는지 알아보도록 하겠습니다. 먼저, favorite television shows 리스트를 비동기로 로드하는 Activity를 만들어 봅시다. 

``` java
Observable<List<String>> tvShowObservable = Observable.fromCallable(new Callable<List<String>>() { 

    @Override 
    public List<String> call() { 
        return mRestClient.getFavoriteTvShows(); 
    }
});
```



이전 예제에서는 Observable을 생성하기 위해 Observable.just() 메서드를 사용했는데요. 이번 예제를 Observable.jus()를 이용하여 구현할 경우 아마 이렇게 될 것 같습니다. 

```java
Observable<List<String>> tvShowObservable = Observable.just(mRestClient.getFavoriteTvShow())
```



하지만, 이전 예제의 getColorList()와 달리 mRestClient.getFavoriteTvShow()는 blocking network call이기 때문에 Observable.just()를 그대로 사용할 경우 Observer가 구독하게 되면 **해당 작업이 처리되는 동안 UI Thread가 blocking되는** 문제가 발생합니다. 



Observable.fromCallable이 생성하는 Observable은 이러한 문제를 해결할 수 있는 중요한 두 가지 특징을 가집니다. 

1. 발행될 value를 생성하는 코드가 Observer에 의해 구독될 때까지 실행되지 않음
2. 생성 코드가 다른 thread에서 실행될 수 있음



이제 Observable을 실제로 구독하는 코드를 보면서 하나씩 알아보도록 하겠습니다.

```java
mTvShowSubscription = tvShowObservable
    .subscribeOn(Schedulers.io())
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe(new Observer<List<String>>() {
        
        @Override 
        public void onCompleted() { }
        
        @Override 
        public void onError(Throwable e) { }

        @Override 
        public void onNext(List<String> tvShows){
            displayTvShows(tvShows);
        }
    });
```

.subscribeOn()은 기본적으로 우리가 생성한 Observable을 다른 thread에서 실행될 수 있는 Observable로 대체합니다. 즉, mRestClient.getFavoriteTvShows()를 포함한 Callable 객체내의 로직은 다른 Thread에서 실행될 수 있다는 얘기가 됩니다. 여기서는 "IO Scheduler"에서 실행되도록 구현이 되어 있습니다. 

이제 Observable의 실행 로직으로 인해 UI Thread가 Blocking되는 일은 없어졌습니다. 하지만, 또 다른 문제가 있는데, Observable이 IO Scheduler에서 실행되기 때문에 이와 상호작용하는 Observer도 IO Scheduler에서 실행됩니다. 즉, onNext()가 onCompleted()가 IO Scheduler에서 실행된다는 말이 됩니다. 

하지만 예제 코드를 보면 onNext()에서 발행된 value를 view에 반영하는 코드가 있는데 view 메서드는 UI Thread에서만 호출될 수 있기 때문에 정상적으로 실행될 수가 없습니다. 

이 문제 또한 간단하게 해결할 수 있는 방법이 있습니다. RxJava에게 Observable을 UI thread에서 observe하라고 알려줄 수가 있습니다. .observeOn() 메서드에 UI Thread를 위한 AndroidSchedules.maintThread()를 명시하면 됩니다. 

이제, blocking task를 실행하는 Observable은 IO Scheduler에서 실행되어 값을 발행하고, 발행된 값을 받는 Observer는 UI Thread에서 실행되어 발행된 값을 UI에 반영하는 것이 가능해졌습니다. 



마지막으로, subscribe 메서드가 생성하는 mTvShowScription에 대해 알아보겠습니다. Observer가 Observable에 구독을 요청하면 Subscription이 생성됩니다. Subscription은 Observer와 Observable 사이의 연결 상태를 나타냅니다. 다른 thread에서 동작 중인 Observable을 구독하고 있는 Activity가 종료될 때 onDestroy()에서는 Subscription을 이용한 아래와 같은 코드가 필요합니다. 

```java
if (mTvShowSubscription != null && !mTvShowSubscription.isUnsubscribed()) {
    mTvShowSubscription.unsubscribe();
}
```

pub/sub 패턴을 사용하게 되면 종종 Activity가 종료되고 나서 다른 thread에서 실행된 결과가 끝이 나는 경우 memory leaks과 NullPointerException을 발생시킬 수 있는데, Subscrition은 unsubscribe()를 이용해 이를 방지할 수 있게 해줍니다. unsubscribe() 메서드를 호출하면 Observer는 더 이상 발행되는 값을 받지 않고, Observable과의 연결이 끊기게 됩니다. 따라서, threading task와 연관된 문제들을 사전에 피할 수 있게 됩니다. 



지금까지 내용을 정리하면,

- Observable.fromCallable()
  - Observable에 의해 발행되는 값의 생성을 실제 구독 시까지 지연시킬 수 있음
  - 코드가 UI Thread가 아닌 다른 thread에서 실행될 수 있음
- subscribeOn()
  - value creation code를 UI Thread가 아닌 특정 thread에서 실행되도록 함
- observeOn()
  - Observable이 발행하는 value를 UI Thread와 같은 적절한 thread에서 지켜볼 수 있도록 함
- Observable이 비동기로 뭔가를 로드했을 때 발생할 수 있는 문제들을 막기 위해 언제든지  Subscription을 unsubscribe할 수 있음



## Example 3: Using Singles

지금까지 예제를 보면 Observable이 유용하다는 것을 충분히 알 수 있지만, 다른 한 편으로 필요 이상으로 과한 경우가 많습니다. 실제로  Example1, 2에서는 모두 value를 한 번 밖에 발행하지 않았고, 따라서 onComplete() 을 사용하지도 않았습니다. 

이런 경우를 위해서 좀 더 간단한 형태인 Observable이 Single을 사용할 수 있습니다. 

Single은 Observable과 거의 동일하게 동작합니다. 단, onComplete(), onNext(), 그리고 onError() 대신 onSuccess()와 onError() 콜백만 있습니다. 



Example2에서의 예제 코드를 Single을 이용해서 다시 구현해보면 아래와 같습니다. 

```java
Single<List<String>> tvShowSingle = Single.fromCallable(new Callable<List<String>>() {

  @Override
  public List<String> call() throws Exception {
    mRestClient.getFavoriteTvShows();
  }
});
```

그리고, Single을 구독해보면

```Java
mTvShowScription = tvShowSingle
  .subscribeOn(Schedulers.io())
  .observeOn(AndroidSchedulers.mainThread())
  .subscribe(new SingleSubscriber<List<String>>() {
    
    @Override
    public void onSuccess(List<String> tvShows) {
      displayTvShows(tvShows);
    }
    
    @Override
    public void onError(Throwable error) {
      displayErrorMessage();
    }
  });
```

SingleSubscriber는 callback이 onSuccess()와 onError() 두 개라는 점만 제외하고는 Observer와 동일하고, Single을 subscribe하는 용도로 사용됩니다. 그리고, 이전 예제에는 없던 error handling 코드가 onError() 콜백에 추가되었습니다. 따라서 mRestClient.getFavoriteTvShows에서 에러가 throws되면, onError() 콜백이 호출됩니다. 



## 마치며

Part1에서는 RxJava(Android)에서 비동기 처리를 제공하는 컨셉인 Observable과 Observer에 대해 이해하고, 이 두 가지를 이용해서 실제 비동기로 데이터를 로드하고, 로드한 결과를 UI에 반영하는 방법까지 알아봤습니다. 가장 관심을 두고 본 내용은 subscribeOn()과 observeOn() 메서를 이용해 비동기 데이터 로드가 UI Thread가 아닌 다른 Thread에서 실행되도록 하는 것과 실행 결과를 UI Thread에서 받을 수 있도록 하는 부분이었습니다. 실제로 안드로이드 앱 개발을 하다보면 이와 같은 구현이 필요한 경우가 많은데 상당히 번거롭고 코드 간결성이나 생산성이 떨어질 뿐만 아니라 실수가 많이 발생할 수 있는 부분입니다. 



아직 RxJava의 기본 컨셉을 이해한 수준이지만, 이번 Part1에서 언급한 장점만으로도 충분히 안드로이드 앱 개발자에게 매력적으로 느껴질 수 있는 라이브리러가 아닌가 하는 생각이 듭니다. 