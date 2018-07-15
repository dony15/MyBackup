





slf4j

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * @author DonY15
 * @description
 * @create 2018\7\14 0014
 */
public class test01 {
    private static final transient Logger log = LoggerFactory.getLogger(test01.class);
    public static void main(String[] args) {
        log.debug("Here is some DEBUG");
        log.info("Here is some INFO");
        log.warn("Here is some WARN");
        log.error("Here is some ERROR");
    }
}
```

