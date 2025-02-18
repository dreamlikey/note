**Hystrix** 是一个用于构建分布式系统中 **容错** 和 **降级** 处理的库，它通过在应用中添加断路器模式来增强系统的稳定性和可靠性。在微服务架构中，服务之间的调用通常是异步的且不可预测的，Hystrix 通过监控服务调用的健康状况，避免了因某个服务的不可用或延迟影响整个系统。

### **Hystrix 实现原理**

Hystrix 的核心思想是 **断路器模式（Circuit Breaker）**，它可以在服务出现故障时迅速进行容错处理。Hystrix 会在服务调用失败达到一定阈值时，触发 **断路器**，从而避免频繁调用导致的系统崩溃，并提供一个备用的降级方案。

#### **主要概念**

1. **断路器模式（Circuit Breaker）**：
   - **闭合状态**：断路器是“闭合”状态时，服务正常调用，如果调用成功，继续正常运行；如果失败，计数器记录失败次数，当失败次数超过阈值时，断路器会进入“打开”状态。
   - **打开状态**：当断路器处于“打开”状态时，所有请求都会被拒绝，直接返回错误或降级结果，不会继续调用底层服务。目的是让底层服务有时间恢复。
   - **半开状态**：当断路器处于“半开”状态时，允许一部分请求通过，用来检测底层服务是否恢复正常。如果请求成功，断路器会重新进入“闭合”状态；如果请求失败，断路器重新进入“打开”状态。
2. **降级（Fallback）**：
   - 当服务调用失败或超时，Hystrix 会调用预先设定的 **降级方法**。降级方法通常是返回一个默认值，或者调用其他备用服务。
3. **超时控制**：
   - Hystrix 对每个请求都设置了 **超时控制**，如果调用时间超过阈值，Hystrix 会自动触发降级处理。
4. **线程隔离（Thread Isolation）**：
   - Hystrix 提供了线程隔离机制。它通过为每个服务调用创建一个独立的线程池来避免线程池耗尽或者某个服务调用失败引发的其他服务调用阻塞。

#### **Hystrix 的工作流程**

1. **请求开始**：
   - 当请求到达一个由 Hystrix 管理的服务时，Hystrix 会首先检查断路器的状态。如果断路器是 **闭合** 的，它会继续发起请求；如果是 **打开** 或 **半开**，它会进行降级处理。
2. **调用执行**：
   - 如果断路器是闭合的，Hystrix 会执行实际的服务调用。Hystrix 可以通过线程隔离或者信号量隔离来控制请求的执行，避免单个服务调用的阻塞影响到其他服务。
3. **请求成功/失败判断**：
   - 如果服务调用成功，Hystrix 会记录成功并继续监控。如果失败（如超时、异常等），Hystrix 会记录失败次数，并决定是否触发断路器。
4. **断路器判断**：
   - 如果服务调用失败达到阈值（比如失败率超过50%），断路器会进入 **打开状态**，从而拒绝所有请求并直接返回降级结果。
   - 如果断路器进入半开状态，Hystrix 会允许一些请求通过，并监控结果来判断服务是否恢复。如果恢复，断路器会进入 **闭合状态**，否则继续保持 **打开状态**。
5. **降级处理**：
   - 当断路器打开或者服务调用超时，Hystrix 会触发降级方法。降级方法可以返回默认值、调用备用服务或者执行其他容错策略。

#### **Hystrix 的核心组成部分**

1. **Command（命令）**：
   - Hystrix 中的每个服务调用都被封装成一个 `HystrixCommand` 对象。命令对象执行具体的业务逻辑，能够自动进行容错处理，如断路器、超时、降级等。
2. **Execution**：
   - Hystrix 提供了两种执行模式：
     - **ThreadPool**：通过线程池隔离服务调用，避免一个调用阻塞线程池中的其他请求。
     - **Semaphore**：通过信号量隔离调用，限制并发请求数量。
3. **Fallback（降级）**：
   - 降级处理是 Hystrix 的核心功能之一。当服务失败时，Hystrix 会调用降级方法（例如返回默认值、备选逻辑等）。
4. **Hystrix Dashboard**：
   - Hystrix 提供了一个仪表盘（Dashboard）来实时监控应用中 Hystrix 命令的执行情况，如请求成功率、失败率、平均响应时间等指标。
5. **Hystrix Stream**：
   - Hystrix Stream 提供了监控系统的实时流式数据，能够收集每个 Hystrix 命令的执行状态并通过 WebSocket 输出到一个 HTML 页面。

### **Hystrix 示例**

#### **1. 创建一个 HystrixCommand**

Hystrix 的使用通常通过继承 `HystrixCommand` 类来创建命令对象。以下是一个示例：

```
java复制编辑import com.netflix.hystrix.HystrixCommand;
import com.netflix.hystrix.HystrixCommandGroupKey;

public class MyHystrixCommand extends HystrixCommand<String> {

    private final String name;

    public MyHystrixCommand(String name) {
        super(HystrixCommandGroupKey.Factory.asKey("ExampleGroup"));
        this.name = name;
    }

    @Override
    protected String run() throws Exception {
        // 执行实际的服务调用逻辑
        if ("fail".equals(name)) {
            throw new RuntimeException("Service call failed");
        }
        return "Hello, " + name;
    }

    @Override
    protected String getFallback() {
        // 执行降级逻辑
        return "Fallback response for " + name;
    }
}
```

#### **2. 使用 HystrixCommand**

```
java复制编辑public class Main {
    public static void main(String[] args) {
        MyHystrixCommand command1 = new MyHystrixCommand("World");
        System.out.println(command1.execute());  // 输出：Hello, World

        MyHystrixCommand command2 = new MyHystrixCommand("fail");
        System.out.println(command2.execute());  // 输出：Fallback response for fail
    }
}
```

在上面的代码中，`MyHystrixCommand` 实现了一个简单的服务调用。`run` 方法表示实际的业务逻辑，如果发生错误或异常，就会触发 `getFallback` 方法进行降级。

#### **3. 设置超时和线程隔离**

```
java复制编辑import com.netflix.hystrix.HystrixCommandProperties;

HystrixCommand.Setter setter = HystrixCommand.Setter
        .withGroupKey(HystrixCommandGroupKey.Factory.asKey("ExampleGroup"))
        .andCommandPropertiesDefaults(
            HystrixCommandProperties.Setter()
                .withExecutionTimeoutInMilliseconds(2000)  // 设置超时
        );

MyHystrixCommand command = new MyHystrixCommand("World");
command.setCommandPropertiesDefaults(setter.getCommandPropertiesDefaults());
System.out.println(command.execute());
```

### **Hystrix 配置**

1. **超时设置：**
   - `hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds`：设置命令执行的超时时间，默认值是 1000 毫秒。
   - `hystrix.command.default.execution.isolation.strategy`：设置命令执行的隔离策略，默认使用线程池（`THREAD`），可以设置为信号量（`SEMAPHORE`）。
2. **断路器设置：**
   - `hystrix.command.default.circuitBreaker.requestVolumeThreshold`：设置断路器的触发阈值，默认值是 20，意味着在 20 次请求内有超过 50% 的失败时，断路器会被打开。
   - `hystrix.command.default.circuitBreaker.sleepWindowInMilliseconds`：设置断路器的睡眠时间，默认值是 5000 毫秒。如果断路器打开，Hystrix 会在此时间段内拒绝所有请求，等待底层服务恢复。
3. **线程池设置：**
   - `hystrix.threadpool.default.coreSize`：设置线程池的核心大小，默认值是 10。
   - `hystrix.threadpool.default.maxQueueSize`：设置线程池队列的最大大小，默认值是 10。

### **总结**

Hystrix 的核心原理是 **断路器模式**，它能够通过对服务调用的隔离、超时控制、失败回退等措施，提升系统的容错性和可用性。Hystrix 使用了 **线程隔离** 和 **信号量隔离** 来避免服务之间的相互阻塞，并提供了可配置的 **超时** 和 **降级策略**。通过 Hystrix，微服务架构能够在面对部分服务故障时，迅速切换到备用逻辑，从而保持系统的整体可用性。





### hystrix 启用了熔断器功能，什么情况下的请求不进行失败成功的统计



###### ChatGPT 说：

在 **Hystrix** 中，启用了熔断器功能后，并不是所有的请求都会被统计为 **成功** 或 **失败**。有一些特殊情况，在这些情况下，Hystrix 并不会将请求计入 **失败** 或 **成功** 的统计中，主要是因为这些请求没有执行真正的 **`run()`** 方法，而是直接进入了 **熔断器短路（circuit breaker short-circuiting）** 或 **`fallback`** 逻辑。

### **请求不进行失败/成功统计的情况：**

1. **熔断器处于 `OPEN` 状态（短路）时的请求**

   当熔断器处于 **`OPEN`** 状态时，所有后续的请求都会被短路。也就是说，这些请求不会进入实际的 **`run()`** 方法，而是直接调用 **`fallback`** 方法。由于请求没有执行正常的业务逻辑，它们不会被计入成功或失败的统计。

   - **`SUCCESS`**：请求没有进入 `run()` 方法，因此不会计入成功。
   - **`FAILURE`**：请求没有进入 `run()` 方法，因此不会计入失败。

   **短路的请求** 会被标记为 **"短路请求"（`short-circuited`）**，而不是 **成功** 或 **失败**。

   #### 触发条件：

   - 熔断器被触发，进入 **`OPEN`** 状态。
   - 当请求量超过阈值并且失败率达到一定比例时，熔断器会触发，后续请求会被 **短路**，直接返回 `fallback`。

2. **`HystrixCommand` 处于 `DISABLED` 或 `FALLBACK` 的情况下**

   如果 **HystrixCommand** 被禁用了（例如通过动态配置或通过代码禁用），或 `fallback` 被手动触发，Hystrix 不会执行 **`run()`** 方法，因此也不会进行失败或成功的统计。在这种情况下，Hystrix 只会记录 **`fallback`** 的行为，但不会记录正常的 **`run()`** 执行结果。

3. **在执行过程中被线程池拒绝的请求（`RejectedExecutionException`）**

   如果 Hystrix 线程池的大小已满，或者信号量池的最大并发请求数已达到上限，新的请求会被拒绝执行。在这种情况下，Hystrix 会记录为 **拒绝**（`Rejected`）而不是 **失败** 或 **成功**。被拒绝的请求不会进入到 `run()` 方法中，因此不会被视为正常的成功或失败请求。

4. **`fallback` 触发时，`fallback` 方法内部抛出异常**

   如果在调用 `fallback` 方法时发生异常（例如，`fallback` 方法本身抛出异常），Hystrix 会记录为 **`FallbackFailure`**，而不是 **`Failure`**。这种情况下，请求会被认为是 **失败的 `fallback`**，而不会被认为是直接的 **失败请求**，因为请求未进入 `run()` 方法。

### **总结：**

在 **Hystrix** 中，只有在请求执行过程中真正进入 **`run()`** 方法并成功执行或失败时，才会被统计为 **`成功`** 或 **`失败`**。

- **短路请求（熔断器处于 `OPEN` 状态）**：请求不会进入 `run()`，不会计入成功或失败统计，记录为 **短路请求**。
- **被线程池或信号量拒绝的请求**：请求不会进入 `run()`，不会计入成功或失败，记录为 **拒绝请求**（`Rejected`）。
- **`fallback` 方法触发时失败**：如果 `fallback` 方法本身抛出异常，Hystrix 会记录为 **`FallbackFailure`**，而不是正常的 **失败**。

这些特殊情况是请求不进行成功/失败统计的原因。





### 当配置了hystrix.command.default.circuitBreaker.forceOpen=true之后，所有请求都只会执行getFallback()方法而不执行run()方法



当配置了 `hystrix.command.default.circuitBreaker.forceOpen=true` 后，熔断器将强制处于 **`OPEN`** 状态，所有的请求都会跳过 **`run()`** 方法，直接调用 **`getFallback()`** 方法。这是 **强制开启熔断器** 的一种特殊配置。

### **`forceOpen` 配置的作用**

- **`hystrix.command.default.circuitBreaker.forceOpen=true`** 强制熔断器进入 **`OPEN`** 状态，不管当前请求的失败率如何，所有后续请求都会被 **短路**，直接调用 `getFallback()` 方法，而不会执行 `run()` 方法。
- 这种配置常用于开发或调试环境中，用来模拟熔断器的 **`OPEN`** 状态，以测试 `fallback` 逻辑或者其他熔断器相关的处理。

### **实际效果**

1. **请求直接执行 `getFallback()`**：无论请求是成功还是失败，熔断器都会跳过 `run()`，直接执行 `getFallback()` 方法。
2. **失败统计不进行**：由于请求没有进入 `run()` 方法，Hystrix 不会对它进行成功或失败的统计。这些请求会被标记为 **短路请求（`short-circuited`）**，而不是普通的失败或成功。

### **代码示例**

```
properties


复制编辑
hystrix.command.default.circuitBreaker.forceOpen=true
```

在这种配置下，所有的请求都会直接触发 **`fallback`**，例如：

```
java复制编辑public class MyHystrixCommand extends HystrixCommand<String> {
    
    public MyHystrixCommand() {
        super(Setter.withGroupKey(HystrixCommandGroupKey.Factory.asKey("ExampleGroup")));
    }

    @Override
    protected String run() {
        // 模拟实际的业务逻辑
        return "Success";
    }

    @Override
    protected String getFallback() {
        // 模拟当熔断器打开时的 fallback 逻辑
        return "Fallback";
    }
}
```

**当 `forceOpen=true` 时**，执行如下代码：

```
java复制编辑MyHystrixCommand command = new MyHystrixCommand();
System.out.println(command.execute()); // Fallback
```

无论 `run()` 方法中的逻辑是否正常，熔断器会 **直接进入 `OPEN` 状态**，请求 **不会执行 `run()`**，而是直接执行 **`getFallback()`**。

### **总结**

- **`forceOpen=true`** 强制熔断器进入 `OPEN` 状态，所有请求都会跳过 `run()`，直接调用 `getFallback()`。
- 这种配置通常用于测试或者故意模拟熔断器的失败场景。