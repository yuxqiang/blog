### NamedContextFactory
public <T> T newInstance(Target<T> target) { 
//修饰了@FeignClient注解的接口方法封装成方法处理器，把指定的target进行解析，得到需要处理的方法集合。
Map<String, MethodHandler> nameToHandler = targetToHandlersByName.apply(target); 
//定义一个用来保存需要处理的方法的集合 
Map<Method, MethodHandler> methodToHandler = new LinkedHashMap<Method, MethodHandler>(); 
//JDK8以后，接口允许默认方法实现，这里是对默认方法进行封装处理。 
List<DefaultMethodHandler> defaultMethodHandlers = new LinkedList<DefaultMethodHandler>();	 
//遍历@FeignClient接口的所有方法 
for (Method method : target.type().getMethods()) { 
//如果是Object中的方法，则直接跳过 
if (method.getDeclaringClass() == Object.class){  
continue;   
} else if (Util.isDefault(method)) {//如果是默认方法，则把该方法绑定一个DefaultMethodHandler。  
DefaultMethodHandler handler = new DefaultMethodHandler(method);   
defaultMethodHandlers.add(handler);   
methodToHandler.put(method, handler); 
} else {//否则，添加MethodHandler(SynchronousMethodHandler)。   
methodToHandler.put(method, nameToHandler.get(Feign.configKey(target.type(), method)));  
} 
} //创建动态代理类。
InvocationHandler handler = factory.create(target, methodToHandler); 
T proxy = (T) Proxy.newProxyInstance(target.type().getClassLoader(),new Class<?>[] {target.type()}, handler); 
for (DefaultMethodHandler defaultMethodHandler : defaultMethodHandlers) {  
defaultMethodHandler.bindTo(proxy); } return proxy;}

