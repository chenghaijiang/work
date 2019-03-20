1、思考并总结装饰者模式和适配器模式的根本区别。

|      | 装饰者模式                                                   | 适配器模式                                                   |
| ---- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 形式 | 是一种非常特别的适配器模式                                   | 没有层级关系，装饰器模式有层级关系                           |
| 定义 | 装饰者和被装饰者都实现同一个接口，主要 目的是为了扩展之后依旧保留OOP关系 | 适配器和被适配者没有必然的联系，通常是采 用继承或代理的形式进行包装 |
| 关系 | 满足is-a的关系                                               | 满足has-a的关系                                              |
| 功能 | 注重覆盖、扩展                                               | 注重兼容、转换                                               |
| 设计 | 前置考虑                                                     | 后置考虑                                                     |

2、用Guava API实现GPer社区提问通知的业务场景

```java
@Data
public class GuavaEvent{
    private String name;
    private String question;
    @Subscribe
    public void subscribe(String str){
        System.out.printf("提问者：%s ,给谁：%s, 提出问题：%s");
    }
}
public class Test{
    public static void main(String[] args){
        EventBus bus = new EventBus();
        GuavaEvent event = new GuavaEvent();
        event.setName("张三");
        event.setQuestion("线程原理是什么？");
        bus.register(event);
        bus.post("tom");
    }
}
```


