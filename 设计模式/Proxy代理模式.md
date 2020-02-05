## 什么是代理

### 静态代理

#### 继承

代理对象继承目标对象，重写目标对象需要增强的方法



#### 聚合

目标对象、代理对象实现同一个接口，代理对象中包含目标对象，代理对象实现增强方法并在方法中调用目标对象的实现。

```java
//目标对象
public interface OrderDao {
    void query();
}

public class OrderDaoImpl implements  OrderDao {
    public void query() {
        System.out.println("----查询订单");
    }
}

//代理对象，增强日志功能
public class OrderDaoLogImpl implements  OrderDao {
	//代理对象中包含目标对象
    private OrderDao orderDao;

    public OrderDaoLogImpl(OrderDao orderDao) {
        this.orderDao = orderDao;
    }

    public void query() {
        System.out.println("----日志");
        orderDao.query();
    }
}

//测试
public class Test {
    public static void main(String[] args) {
        OrderDao target = new OrderDaoImpl();
        OrderDao proxy = new OrderDaoLogImpl(target);
        proxy.query();
    }
}

```



### 动态代理

