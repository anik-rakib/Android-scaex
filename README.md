# Android-scaex

[![Circle CI](https://circleci.com/gh/petitviolet/Android-scaex.svg?style=svg)](https://circleci.com/gh/petitviolet/Android-scaex)
[![Download](https://api.bintray.com/packages/petitviolet/maven/android-scaex/images/download.svg)](https://bintray.com/petitviolet/maven/android-scaex/\_latestVersion)
[![Android Arsenal](https://img.shields.io/badge/Android%20Arsenal-Android--scaex-green.svg?style=true)](https://android-arsenal.com/details/1/2859)

This library provides some expressions, `IF`, `Match` like Scala.  
`IF` enables java if-statement to return a value as Scala if-expression.  
`Match` also enables java to use pattern-match expression.

# Setup

```groovy
dependencies {
    compile 'net.petitviolet.android:scaex:<latest-version>
}
```

# Interface

### IF

```java
/** returns primitive value pattern **/
// if ~ else if ~ else
String result = IF.<String>x(false).then("hoge")
        .ElseIf(false).then("foo")
        .Else("bar");
assert result == "bar";

// if ~ else if ~
String result2 = IF.<String>x(true).then("hoge")
        .ElseIf(false).then("foo")
        .get();
assert result2 == "hoge";

// not matched
String result3 = IF.<String>x(false).then("hoge").get()
assert result3 == null;


/** returns value the result of given method invoked **/
String result4 = IF.<String>x(false).then(new Action<String>() {
    @Override
    public String run() {
        return "hoge";
    }
}).ElseIf(true).then(new Action<String>() {
    @Override
    public String run() {
        return "foo";
    }
}).Else(new Action<String>() {
    @Override
    public String run() {
        Log.d(TAG, "in else condition!");
        return "bar";
    }
});

assert result4 == "foo";

// with lambda expression
String result5 = IF.<String>x(false).then(() -> "hoge")
    .ElseIf(true).then(() -> "foo")
    .Else(() -> {
        Log.d(TAG, "in else condition!");
        return "bar";
    });

assert result5 == "foo";
```

### Match

```java
/** pattern match by value condition **/
int target = 100;
String result1 = Match.<String, Integer>x(target)
        .Case(target < 100).then("down")
        .Case(target > 100).then("up")
        .Case(target == 100).then("draw")
        .eval();
assert result1.equals("draw") == true;

/** pattern match by class **/
int result2 = Match.<Integer, Object>x(100)
		.Case(String.class).then(1)
		.Case(Boolean.class).then(2)
		.Case(Integer.class).then(3)
		.eval();
assert result2 == 3;

/** lazy pattern-match using Function interface in Case **/
int result3 = Match.<Integer, Object>x(100)
		.Case(String.class).then(1)
		.Case(Boolean.class).then(2)
		.Case(new Function.F1<Object, Boolean>() {
			@Override
			public Boolean apply(Object integer) {
				return integer == 100;
			}
		}).then(3)
		.Case(new Function.F1<Object, Boolean>() {
			@Override
			public Boolean apply(Object integer) {
                // not evaluate
				return integer == 999;
			}
		}).then(4)
		.Case(Integer.class).then(5)
		.eval();
assert result3 == 3;

/** with lambda expression **/
String result4 = Match.<String, String>x("hoge")
        .Case(TextUtils::isEmpty).then("empty")
        .Case((s) -> s.length() < 3).then("short")
        .eval("long");
assert result4 == "long";
```

### For

```java
// for(Integer i: inputs) { i -> action(i) }
List<Integer> inputs = Arrays.asList(1, 2, 3);
For.x(inputs).action(new Action.A1<Integer>() {
    @Override
    public void run(Integer integer) {
        Log.i(Tag, s"result ->" + integer * 2);
    }
});

// for statement with returning value
// result = for(String s: strings) { s -> function(s) }
List<String> result = For.x(Arrays.asList("a", "b", "c")).apply(new Function.F1<String, String>() {
    @Override
    public String apply(String s) {
        return "x" + s;
    }
});

// for(int i = 0; i <= 10; i++) { // }
// `x(0).to(10).by(1)`
List<Integer> result = For.x(0).to(10).by(1).apply(new Function.F1<Integer, Integer>() {
    @Override
    public Integer apply(Integer integer) {
        return integer;
    }
});

// for(int i = 1; i < 10; i = i + 2 { // }
// `x(1).until(10).by(2)`
List<Integer> result = For.x(1).until(10).by(2).apply(new Function.F1<Integer, Integer>() {
    @Override
    public Integer apply(Integer integer) {
        return integer;
    }
});


// example using lambda
// like foreach
For.x(inputs).action(s -> Log.i(String.format("testFor: %s", input), s));
// like map
List<String> results = For.x(inputs).apply(s -> String.format("applied: %s: %s", input, s));
```

# Lisence

This code is licensed under the Apache Software License 2.0.
