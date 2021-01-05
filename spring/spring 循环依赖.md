spring 循环依赖

简易代码逻辑

```java
    public static Object create(Class obj) throws IllegalAccessException, InstantiationException, NoSuchMethodException, InvocationTargetException {

        if (!map.containsKey(obj.getName())) {
            Object o = obj.newInstance();
            map.put(obj.getName(), o);
        }

        Object o = map.get(obj.getName());
        Field[] fields = o.getClass().getDeclaredFields();

        for (Field field : fields) {
            if (!map.containsKey(field.getType().getName())) {

                create(field.getType());
            }

            String fieldName = field.getType().getName();

            String setMethodName = "set" + fieldName.toUpperCase();
            Method method = obj.getDeclaredMethod(setMethodName, field.getType());
            method.setAccessible(true);
            method.invoke(o, map.get(field.getType().getName()));
        }
        
        return o;
    }
```



