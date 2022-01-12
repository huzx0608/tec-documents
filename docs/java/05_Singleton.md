# Singleton

## Implement by Double-check

```Java
public class Singleton {
    private static volatile Singleton instance;

    private Singleton() {}

    public static Singleton getInstance() {
       if (null == instance) {
          synchronized (Singleton.class) {
              if (null == instance) {
                  instance = new Singleton();
              }
          }
       }

       return Singleton.instance;
    }
};
```


## Implement by class holder

public class Singleton {

   private Singleton() {}
   private static class SingletonHolder {
      private static final Singleton instance = new Singleton();
   }

   public static Singleton getInstance() {
     return SingletonHolder.instance;
   }
}

## Implement by Enum class

public class Singleton {
  private Singleton() {}

  private enum InnerSingle {
     INSTANCE;

     private final Singleton singleInstance;

     InnerSingle() {
        singleInstance = new Singleton();
     }

     public Singleton getInstance() {
       return singleInstance;
     }
  }

  public static Singleton getInstance() {
      return InnerSingle.INSTANCE.getInstance();
  }
}
