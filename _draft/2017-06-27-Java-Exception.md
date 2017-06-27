Question: Is exception handling in Java actually slow?

The simplest way is using setjmp and longjmp. That means all registers of the CPU are written to the stack (which already takes some time) and possibly some other data needs to be created... all this already happens in the try statement. The throw statement needs to unwind the stack and restore the values of all registers (and possible other values in the VM). So try and throw are equally slow, and that is pretty slow, however if no exception is thrown, exiting the try block takes no time whatsoever in most cases (as everything is put on the stack which cleans up automatically if the method exists).

Sun and others recognized, that this is possibly suboptimal and of course VMs get faster and faster over the time. There is another way to implement exceptions, which makes try itself lightning fast (actually nothing happens for try at all in general - everything that needs to happen is already done when the class is loaded by the VM) and it makes throw not quite as slow. I don't know which JVM uses this new, better technique...

...but are you writing in Java so your code later on only runs on one JVM on one specific system? Since if it may ever run on any other platform or any other JVM version (possibly of any other vendor), who says they also use the fast implementation? The fast one is more complicated than the slow one and not easily possible on all systems. You want to stay portable? Then don't rely on exceptions being fast.

It also makes a big difference what you do within a try block. If you open a try block and never call any method from within this try block, the try block will be ultra fast, as the JIT can then actually treat a throw like a simple goto. It neither needs to save stack-state nor does it need to unwind the stack if an exception is thrown (it only needs to jump to the catch handlers). However, this is not what you usually do. Usually you open a try block and then call a method that might throw an exception, right? And even if you just use the try block within your method, what kind of method will this be, that does not call any other method? Will it just calculate a number? Then what for do you need exceptions? There are much more elegant ways to regulate program flow. For pretty much anything else but simple math, you will have to call an external method and this already destroys the advantage of a local try block.

See the following test code:

public class Test {
    int value;


    public int getValue() {
        return value;
    }

    public void reset() {
        value = 0;
    }

    // Calculates without exception
    public void method1(int i) {
        value = ((value + i) / i) << 1;
        // Will never be true
        if ((i & 0xFFFFFFF) == 1000000000) {
            System.out.println("You'll never see this!");
        }
    }

    // Could in theory throw one, but never will
    public void method2(int i) throws Exception {
        value = ((value + i) / i) << 1;
        // Will never be true
        if ((i & 0xFFFFFFF) == 1000000000) {
            throw new Exception();
        }
    }

    // This one will regularly throw one
    public void method3(int i) throws Exception {
        value = ((value + i) / i) << 1;
        // i & 1 is equally fast to calculate as i & 0xFFFFFFF; it is both
        // an AND operation between two integers. The size of the number plays
        // no role. AND on 32 BIT always ANDs all 32 bits
        if ((i & 0x1) == 1) {
            throw new Exception();
        }
    }

    public static void main(String[] args) {
        int i;
        long l;
        Test t = new Test();

        l = System.currentTimeMillis();
        t.reset();
        for (i = 1; i < 100000000; i++) {
            t.method1(i);
        }
        l = System.currentTimeMillis() - l;
        System.out.println(
            "method1 took " + l + " ms, result was " + t.getValue()
        );

        l = System.currentTimeMillis();
        t.reset();
        for (i = 1; i < 100000000; i++) {
            try {
                t.method2(i);
            } catch (Exception e) {
                System.out.println("You'll never see this!");
            }
        }
        l = System.currentTimeMillis() - l;
        System.out.println(
            "method2 took " + l + " ms, result was " + t.getValue()
        );

        l = System.currentTimeMillis();
        t.reset();
        for (i = 1; i < 100000000; i++) {
            try {
                t.method3(i);
            } catch (Exception e) {
                // Do nothing here, as we will get here
            }
        }
        l = System.currentTimeMillis() - l;
        System.out.println(
            "method3 took " + l + " ms, result was " + t.getValue()
        );
    }
}

Result:

method1 took 972 ms, result was 2
method2 took 1003 ms, result was 2
method3 took 66716 ms, result was 2
The slowdown from the try block is too small to rule out confounding factors such as background processes. But the catch block killed everything and made it 66 times slower!

As I said, the result will not be that bad if you put try/catch and throw all within the same method (method3), but this is a special JIT optimization I would not rely upon. And even when using this optimization, the throw is still pretty slow. So I don't know what you are trying to do here, but there is definitely a better way of doing it than using try/catch/throw.


