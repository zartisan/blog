---
title: Flink KeyedStream Aggregations
---


## 官方文档
Rolling aggregations on a keyed data stream. The difference between min and minBy is that min returns the minimum value, whereas minBy returns the element that has the minimum value in this field (same for max and maxBy).
>[文档链接](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/stream/operators/)

```java
keyedStream.sum(0);
keyedStream.sum("key");
keyedStream.min(0);
keyedStream.min("key");
keyedStream.max(0);
keyedStream.max("key");
keyedStream.minBy(0);
keyedStream.minBy("key");
keyedStream.maxBy(0);
keyedStream.maxBy("key");
```

问题是min()和minBy()区别到底是什么，官方文档说的不太详细，只能研究源码。

## KeyedStream
Aggregations的相关方法如下：

```java
	public SingleOutputStreamOperator<T> sum(int positionToSum) {
		return aggregate(new SumAggregator<>(positionToSum, getType(), getExecutionConfig()));
	}


	public SingleOutputStreamOperator<T> sum(String field) {
		return aggregate(new SumAggregator<>(field, getType(), getExecutionConfig()));
	}


	public SingleOutputStreamOperator<T> min(int positionToMin) {
		return aggregate(new ComparableAggregator<>(positionToMin, getType(), AggregationFunction.AggregationType.MIN,
				getExecutionConfig()));
	}


	public SingleOutputStreamOperator<T> min(String field) {
		return aggregate(new ComparableAggregator<>(field, getType(), AggregationFunction.AggregationType.MIN,
				false, getExecutionConfig()));
	}


	public SingleOutputStreamOperator<T> max(int positionToMax) {
		return aggregate(new ComparableAggregator<>(positionToMax, getType(), AggregationFunction.AggregationType.MAX,
				getExecutionConfig()));
	}


	public SingleOutputStreamOperator<T> max(String field) {
		return aggregate(new ComparableAggregator<>(field, getType(), AggregationFunction.AggregationType.MAX,
				false, getExecutionConfig()));
	}


	@SuppressWarnings({ "rawtypes", "unchecked" })
	public SingleOutputStreamOperator<T> minBy(String field, boolean first) {
		return aggregate(new ComparableAggregator(field, getType(), AggregationFunction.AggregationType.MINBY,
				first, getExecutionConfig()));
	}


	public SingleOutputStreamOperator<T> maxBy(String field, boolean first) {
		return aggregate(new ComparableAggregator<>(field, getType(), AggregationFunction.AggregationType.MAXBY,
				first, getExecutionConfig()));
	}


	public SingleOutputStreamOperator<T> minBy(int positionToMinBy) {
		return this.minBy(positionToMinBy, true);
	}


	public SingleOutputStreamOperator<T> minBy(String positionToMinBy) {
		return this.minBy(positionToMinBy, true);
	}


	public SingleOutputStreamOperator<T> minBy(int positionToMinBy, boolean first) {
		return aggregate(new ComparableAggregator<T>(positionToMinBy, getType(), AggregationFunction.AggregationType.MINBY, first,
				getExecutionConfig()));
	}


	public SingleOutputStreamOperator<T> maxBy(int positionToMaxBy) {
		return this.maxBy(positionToMaxBy, true);
	}


	public SingleOutputStreamOperator<T> maxBy(String positionToMaxBy) {
		return this.maxBy(positionToMaxBy, true);
	}


	public SingleOutputStreamOperator<T> maxBy(int positionToMaxBy, boolean first) {
		return aggregate(new ComparableAggregator<>(positionToMaxBy, getType(), AggregationFunction.AggregationType.MAXBY, first,
				getExecutionConfig()));
	}
```

> 每个sum、max、min、maxBy、minBy都有两个重载方法，一个是int类型的参数，表示是第几个field，一个是String类型的参数，表示具体的field名称；
> maxBy、minBy比sum、max、min多了first(boolean)参数，该参数用于指定在碰到多个compare值相等时，是否取第一个返回，默认为true。
> min、minBy、max、maxBy都是新建`new ComparableAggregator<>()`对象并作为参数传递给`aggregate()`函数。

## ComparableAggregator
源码如下：

```java
@Internal
public class ComparableAggregator<T> extends AggregationFunction<T> {

	private static final long serialVersionUID = 1L;

	private Comparator comparator;
	private boolean byAggregate;
	private boolean first;
	private final FieldAccessor<T, Object> fieldAccessor;

	private ComparableAggregator(AggregationType aggregationType, FieldAccessor<T, Object> fieldAccessor, boolean first) {
		this.comparator = Comparator.getForAggregation(aggregationType);
		this.byAggregate = (aggregationType == AggregationType.MAXBY) || (aggregationType == AggregationType.MINBY);
		this.first = first;
		this.fieldAccessor = fieldAccessor;
	}

	public ComparableAggregator(int positionToAggregate,
			TypeInformation<T> typeInfo,
			AggregationType aggregationType,
			ExecutionConfig config) {
		this(positionToAggregate, typeInfo, aggregationType, false, config);
	}

	public ComparableAggregator(int positionToAggregate,
			TypeInformation<T> typeInfo,
			AggregationType aggregationType,
			boolean first,
			ExecutionConfig config) {
		this(aggregationType, FieldAccessorFactory.getAccessor(typeInfo, positionToAggregate, config), first);
	}

	public ComparableAggregator(String field,
			TypeInformation<T> typeInfo,
			AggregationType aggregationType,
			boolean first,
			ExecutionConfig config) {
		this(aggregationType, FieldAccessorFactory.getAccessor(typeInfo, field, config), first);
	}

	@SuppressWarnings("unchecked")
	@Override
	public T reduce(T value1, T value2) throws Exception {
		Comparable<Object> o1 = (Comparable<Object>) fieldAccessor.get(value1);
		Object o2 = fieldAccessor.get(value2);

		int c = comparator.isExtremal(o1, o2);

		if (byAggregate) {
			// if they are the same we choose based on whether we want to first or last
			// element with the min/max.
			if (c == 0) {
				return first ? value1 : value2;
			}

			return c == 1 ? value1 : value2;

		} else {
			if (c == 0) {
				value1 = fieldAccessor.set(value1, o2);
			}
			return value1;
		}

	}

}
```

```java
@Internal
public abstract class Comparator implements Serializable {

	private static final long serialVersionUID = 1L;

	public abstract <R> int isExtremal(Comparable<R> o1, R o2);

	public static Comparator getForAggregation(AggregationType type) {
		switch (type) {
		case MAX:
			return new MaxComparator();
		case MIN:
			return new MinComparator();
		case MINBY:
			return new MinByComparator();
		case MAXBY:
			return new MaxByComparator();
		default:
			throw new IllegalArgumentException("Unsupported aggregation type.");
		}
	}

	private static class MaxComparator extends Comparator {

		private static final long serialVersionUID = 1L;

		@Override
		public <R> int isExtremal(Comparable<R> o1, R o2) {
			return o1.compareTo(o2) > 0 ? 1 : 0;
		}

	}

	private static class MaxByComparator extends Comparator {

		private static final long serialVersionUID = 1L;

		@Override
		public <R> int isExtremal(Comparable<R> o1, R o2) {
			int c = o1.compareTo(o2);
			if (c > 0) {
				return 1;
			}
			if (c == 0) {
				return 0;
			} else {
				return -1;
			}
		}

	}

	private static class MinByComparator extends Comparator {

		private static final long serialVersionUID = 1L;

		@Override
		public <R> int isExtremal(Comparable<R> o1, R o2) {
			int c = o1.compareTo(o2);
			if (c < 0) {
				return 1;
			}
			if (c == 0) {
				return 0;
			} else {
				return -1;
			}
		}

	}

	private static class MinComparator extends Comparator {

		private static final long serialVersionUID = 1L;

		@Override
		public <R> int isExtremal(Comparable<R> o1, R o2) {
			return o1.compareTo(o2) < 0 ? 1 : 0;
		}

	}
}
```

>构造函数会根据不同的AggregationType(MAX/MIN/MINBY/MAXBY)构造不同的Comparator对象。主要区别是MINBY/MAXBY对比会返回-1，0，1，而MIN/MAX返回0，1。
>reduce函数就是问题的答案，以max和maxBy为例，其逻辑如下：

1. 如果是maxBy，如果相等，first为true返回第一个value；first为false返回第二个value；
2. 如果是max，如果value1比value2大，返回value1；否则，将value2中用来作对比的field的值替换value1中该field的值，并返回value1。

简而言之，maxBy返回的是field最大的那个element，max是将最大的field赋值给第一个element并返回第一个element。


## 测试
### POJO


```java
public class MyWordCount {
    private int count;
    private String word;
    private int frequency;

    public MyWordCount() {
    }

    public MyWordCount(int count, String word, int frequency) {
        this.count = count;
        this.word = word;
        this.frequency = frequency;
    }

    public int getCount() {
        return count;
    }

    public void setCount(int count) {
        this.count = count;
    }

    public String getWord() {
        return word;
    }

    public void setWord(String word) {
        this.word = word;
    }

    public int getFrequency() {
        return frequency;
    }

    public void setFrequency(int frequency) {
        this.frequency = frequency;
    }

    @Override
    public String toString() {
        return "MyWordCount{" +
            "count=" + count +
            ", word='" + word + '\'' +
            ", frequency=" + frequency +
            '}';
    }
}
```

### 测试代码：

```java
public class DataStreamTest {

    private MyWordCount[] data = new MyWordCount[]{
        new MyWordCount(1,"Hello", 1),
        new MyWordCount(2,"Hello", 2),
        new MyWordCount(3,"Hello", 3),
        new MyWordCount(1,"World", 3)
    };

    @Test
    public void testMax() throws Exception {
        final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.fromElements(data)
            .keyBy("word")
            .max("frequency")
            .addSink(new SinkFunction<MyWordCount>() {
                @Override
                public void invoke(MyWordCount value, Context context) {
                    System.err.println("\n" + value + "\n");
                }
            });
        env.execute("testMax");
    }

    @Test
    public void testMaxBy() throws Exception {
        final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.fromElements(data)
            .keyBy("word")
            .maxBy("frequency")
            .addSink(new SinkFunction<MyWordCount>() {
                @Override
                public void invoke(MyWordCount value, Context context) {
                    System.err.println("\n" + value + "\n");
                }
            });
        env.execute("testMax");
    }
}
```

### 结果
testMax:
> MyWordCount{count=1, word='World', frequency=3}
> MyWordCount{count=1, word='Hello', frequency=1}
> MyWordCount{count=1, word='Hello', frequency=2}
> MyWordCount{count=1, word='Hello', frequency=3}

testMaxBy:
> MyWordCount{count=1, word='World', frequency=3}
> MyWordCount{count=1, word='Hello', frequency=1}
> MyWordCount{count=2, word='Hello', frequency=2}
> MyWordCount{count=3, word='Hello', frequency=3}


