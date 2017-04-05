### Scala重写Java可变参数方法问题 ###

```Java
public interface KeyGenerator {
	/**
	 * Generate a key for the given method and its parameters.
	 * @param target the target instance
	 * @param method the method being called
	 * @param params the method parameters (with any var-args expanded)
	 * @return a generated key
	 */
	Object generate(Object target, Method method, Object... params);
}
```

如以上的Java接口，Java中的可变参数是以...出现，而Scala则不同，Scala的可变参数是以*出现的，如下：

```Scala
def generate(target:Object, method: Method, params: Object*): Object = {
		//省略
}
```

如果你使用IDE自动生成的方法参数如下：

```Scala
def generate(target:Any, method: Method, params: Array[AnyRef]): Object = {
		//省略
}
```

在使用Scala重写了KeyGenerator接口的generate方法时编译器总是无法通过，即在Scala中使用params: Object*或者是Array[AnyRef]是无法替代Object... params的，注意Scala中的Any和AnyRef都是Object类型，classOf[Any]和classOf[AnyRef]都是Object类型,解决办法是你只需要将Array[AnyRef]修改成AnyRef*就可以编译通过了。

```Scala
def wiselyKeyGenerator(): KeyGenerator = {
    new KeyGenerator() {
        override protected def generate(target: Any, method: Method, params: AnyRef*): Object = {
              var sb = new StringBuilder()
              sb.append(target.getClass().getName())
              sb.append(method.getName())
              for(param <- params) {
                  sb.append(param.toString())
              }
              sb.toString()
          }
    };
}
```