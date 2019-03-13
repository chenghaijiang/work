1、运用原型模式重构一段业务代码。

**重构前的代码**

```java
@Data
public class FormVo {
    Integer id；
    String name；  
}
@Data
public class FormEntity {
    Integer id;
    String name;
}
@RestController
public class PlanController{
    @Autowire
    PlanMapper mapper;
    @PostMapping("add")
    ResponseEntity add(FormVo vo){
        FormEntity entity = new FormEntity();
        entity.setId(vo.getId());
        entity.setName(vo.getName());
        mapper.insert(entity);
        return ResponseEntity.ok("新增成功");
    }
}
```

**重构后的代码**

```java
@RestController
public class PlanController{
    @Autowire
    PlanMapper mapper;
    @PostMapping("add")
    ResponseEntity add(FormVo vo){
        FormEntity entity = new FormEntity();
        BeanUtils.copyProperties(vo,entity);
        mapper.insert(entity);
        return ResponseEntity.ok("新增成功");
    }
}
```

