+++
date = "2016-09-24T20:49:02+02:00"
title = "Yield or not to yield"
categories = ["c#"]
+++

.Net community have frozen in anticipation of C# 7.0 and new features that it brings. Each version of the language that will turn 15 years old next year brought something new and useful for developers. Though each feature worth separate mention, today I want to tell you about keyword *yield*. I noticed, that beginners (not only) avoid using it. In this article I'll try to show its pros and cons, and provide cases when *yield* usage makes sense.

*yield* creates iterator and lets us do not write separate `Enumerator` class when we implement `IEnumerable`. C# has two expressions that contains *yield*: `yield return <expression>` и *yield break*. *yield* can be used in methods, operators or get accessors but I'll mostly talk about methods as *yield* works the same way everywhere.

Writing *yield return* we indicate that current method returns `IEnumerable`, which elements are results of *yield return* expressions. After *yield* method stops its execution it returns control to caller. *yield* continues execution after next element of sequence is requested. Method variables retains their values between *yield return* expressions. *yield break* does well-known role of *break* which we use in loops. Example below returns numbers sequence from 0 to 10:

```
private static IEnumerable<int> GetNumbers() {
    var number = 0;
    while (true) {
        if (number > 10)
            yield break;

        yield return number++;
    }
}
```

It's important to notice that *yield* has several constraints. Iterator's `Reset` method throws `NotSupportedException`. We can't use *yield* in anonymous methods and methods that have `unsafe` code. Also, *yield return* can't be placed inside try-catch, but can be used in `try` section of `try-finally`. *yield break* can be placed in section `try` of both `try-catch` and `try-finally`. I recommend to read about reasons of such behavior [here](https://blogs.msdn.microsoft.com/ericlippert/2009/07/16/iterator-blocks-part-three-why-no-yield-in-finally/) and [here](https://blogs.msdn.microsoft.com/ericlippert/2009/07/20/iterator-blocks-part-four-why-no-yield-in-catch/).
<!--more-->
Let’s see what *yield* compiles into. Each method with *yield return* is represented by a state machine which goes from one state to another during iterator execution. Below is listed simple application which prints infinite sequence of odd numbers:

```
internal class Program
{
    private static void Main() {
        foreach (var number in GetOddNumbers())
            Console.WriteLine(number);
    }

    private static IEnumerable<int> GetOddNumbers() {
        var previous = 0;
        while (true)
            if (++previous%2 != 0)
                yield return previous;
    }
}
```

Compiler generates next code:

```
internal class Program
{
    private static void Main() {
        IEnumerator<int> enumerator = null;
        try {
            enumerator = GetOddNumbers().GetEnumerator();
            while (enumerator.MoveNext())
                Console.WriteLine(enumerator.Current);
        } finally {
            if (enumerator != null)
                enumerator.Dispose();
        }
    }

    [IteratorStateMachine(typeof(CompilerGeneratedYield))]
    private static IEnumerable<int> GetOddNumbers() {
        return new CompilerGeneratedYield(-2);
    }

    [CompilerGenerated]
    private sealed class CompilerGeneratedYield : IEnumerable<int>, 
        IEnumerable, IEnumerator<int>, IDisposable, IEnumerator
    {
        private readonly int _initialThreadId;
        private int _current;
        private int _previous;
        private int _state;

        [DebuggerHidden]
        public CompilerGeneratedYield(int state) {
            _state = state;
            _initialThreadId = Environment.CurrentManagedThreadId;
        }

        [DebuggerHidden]
        IEnumerator<int> IEnumerable<int>.GetEnumerator() {
            CompilerGeneratedYield getOddNumbers;
            if ((_state == -2) && (_initialThreadId == Environment.CurrentManagedThreadId)) {
                _state = 0;
                getOddNumbers = this;
            } else {
                getOddNumbers = new CompilerGeneratedYield(0);
            }

            return getOddNumbers;
        }

        [DebuggerHidden]
        IEnumerator IEnumerable.GetEnumerator() {
            return ((IEnumerable<int>)this).GetEnumerator();
        }

        int IEnumerator<int>.Current {
            [DebuggerHidden] get { return _current; }
        }

        object IEnumerator.Current {
            [DebuggerHidden] get { return _current; }
        }

        [DebuggerHidden]
        void IDisposable.Dispose() { }

        bool IEnumerator.MoveNext() {
            switch (_state) {
                case 0:
                    _state = -1;
                    _previous = 0;
                    break;
                case 1:
                    _state = -1;
                    break;
                default:
                    return false;
            }

            int num;
            do {
                num = _previous + 1;
                _previous = num;
            } while (num%2 == 0);

            _current = _previous;
            _state = 1;

            return true;
        }

        [DebuggerHidden]
        void IEnumerator.Reset() {
            throw new NotSupportedException();
        }
    }
}
```

From example you can find, that *yield* method body was replaced with class which implements `IEnumerable` and `IEnumerator`. It has *yield* method local variables as fields. Method's logic was transformed into a state machine and moved to `MoveNext`. Depending on initial *yield* logic class can have `Dispose` method implementation.

Let’s go further and do 2 tests to measure *yield* performance and memory consumption. Just to note - these tests are synthetic and listed here only to show *yield* in comparison with straight implementation. I'm using [BenchmarkDotNet](https://github.com/PerfDotNet/BenchmarkDotNet) with `BenchmarkDotNet.Diagnostics.Windows` diagnostic module. First test compares implementations of method that returns sequence of integers (like `Enumerable.Range(start, count)`). First implementation is without iterator, second with:

```
public int[] Array(int start, int count) {
    var numbers = new int[count];
    for (var i = 0; i < count; ++i)
        numbers[i] = start + i;

    return numbers;
}

public int[] Iterator(int start, int count) {
    return IteratorInternal(start, count).ToArray();
}

private IEnumerable<int> IteratorInternal(int start, int count) {
    for (var i = 0; i < count; ++i)
        yield return start + i;
}
```

   Method | Count | Start |      Median |   StdDev |    Gen 0 | Gen 1 | Gen 2 | Bytes Allocated/Op
--------- |------ |------ |------------ |--------- |--------- |------ |------ |-------------------
    Array |   100 |    10 |    91.19 ns |  1.25 ns |   385.01 |     - |     - |             169.18
 Iterator |   100 |    10 | 1,173.26 ns | 10.94 ns | 1,593.00 |     - |     - |             700.37

As you can see from results, Array implementation is almost 10 times faster and consumes almost 4 times less memory. Iterator class and separate `ToArray` call do their job.

Second test is more complex. It emulates data stream processing. It sequentially selects records with even key and then records with key multiplied by 3. Similar to previous test, first implementation is without iterator, second with:

```
public List<Tuple<int, string>> List(int start, int count) {
    var odds = new List<Tuple<int, string>>();
    foreach (var record in OddsArray(ReadFromDb(start, count)))
        if (record.Item1%3 == 0)
            odds.Add(record);

    return odds;
}

public List<Tuple<int, string>> Iterator(int start, int count) {
    return IteratorInternal(start, count).ToList();
}

private IEnumerable<Tuple<int, string>> IteratorInternal(int start, int count) {
    foreach (var record in OddsIterator(ReadFromDb(start, count)))
        if (record.Item1%3 == 0)
            yield return record;
}

private IEnumerable<Tuple<int, string>> OddsIterator(IEnumerable<Tuple<int, string>> records) {
    foreach (var record in records)
        if (record.Item1%2 != 0)
            yield return record;
}

private List<Tuple<int, string>> OddsArray(IEnumerable<Tuple<int, string>> records) {
    var odds = new List<Tuple<int, string>>();
    foreach (var record in records)
        if (record.Item1%2 != 0)
            odds.Add(record);

    return odds;
}

private IEnumerable<Tuple<int, string>> ReadFromDb(int start, int count) {
    for (var i = start; i < count; ++i)
        yield return new KeyValuePair<int, string>(start + i, RandomString());
}

private static string RandomString() {
    return Guid.NewGuid().ToString("n");
}
```

   Method | Count | Start |   Median |  StdDev |  Gen 0 | Gen 1 | Gen 2 | Bytes Allocated/Op
--------- |------ |------ |--------- |-------- |------- |------ |------ |-------------------
     List |   100 |    10 | 43.14 us | 0.14 us | 279.04 |     - |     - |           4,444.14
 Iterator |   100 |    10 | 43.22 us | 0.76 us | 231.00 |     - |     - |           3,760.96

In this test, implementations performance is the same, but *yield* memory consumption is lower. It's due to the fact that collection is computed only once and we saved memory on allocation only one `List<Tuple<int, string>>`.

Considering all the above, I can do a short conclusion. Main disadvantage of *yield* is creation of   additional iterator class. When sequence is finite and caller doesn't have complex logic *yield* is slower and allocates more memory. Usage of *yield* makes sense in cases of data processing when each collection computation causes large memory block allocation. Lazy nature of *yield* can help to avoid computation of elements that will be filtered. In such cases *yield* significantly reduces memory consumption and CPU load.