1、思考：模板模式除了继承以外，还有哪些实现方式？

动态代理

2、使用适配模式，重构一段需要升级功能且兼容老系统的业务代码。

```java
public class PayService{
    public void pay(){
        System.out.println("支付");
    }
}
```

```java
public class AliPayService extends PayService{
    public void aliPay(){
        System.out.println("阿里");
        this.pay();
    }
}
```


