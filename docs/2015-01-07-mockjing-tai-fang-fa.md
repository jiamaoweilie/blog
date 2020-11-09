# 如何Mock静态方法

最近为客户一堆没有单元测试的代码补测试时，遇到了一堆奇葩的问题，其中一个是被测函数需要调用另外一个类中的静态方法。今天看几个同事讨论说这种写法不好，但是whatever，谁又能完全在理想状态下生存，现有的系统就是这样，测试总得写的吧。

那么我们的解决方法就是使用PowerMock将这些静态方面mock掉。怎么做的呢，通过下面的小例子说明一下。

假如我们有这样一个类EmployeeService需要测试：

``` java
public class EmployeeService {
    public int getEmployeeCount() {
        return Employee.count();
    }
}
```

可以看出，这个类中有一个函数`getEmployeeCount()`，它里面调用了类`Employee`中的`count()`函数。需要测试的类是`EmployeeService`，那么自然不应该调用真实的类`Employee`。如何mock该类呢，我们可以使用PowerMock。

测试的实现代码如下：

``` java
@RunWith(PowerMockRunner.class)
@PrepareForTest(Employee.class)
public class EmployeeServiceTest {

   @Test
   public void shouldReturnTheCountOfEmployeesUsingTheDomainClass() {
      PowerMockito.mockStatic(Employee.class);
      PowerMockito.when(Employee.count()).thenReturn(900);

      EmployeeService employeeService = new EmployeeService();
      assertEquals(900,employeeService.getEmployeeCount());
   }
}
```

可以看到，测试代码的顶部使用了两个annotation（ `@RunWith` 和 `@PrepareForTest` ），在测试实现中使用了 `PowerMockito` 。其中 `@RunWith(PowerMockRunner.class)` 告诉JUnit使用PowerMockRunner执行该测试。`@PrepareForTest(Employee.class)`告诉PowerMock类 `Employee` 是测试中需要用到的类。当我们需要mock一个final类，或需要mock的类中有final、static、private方法时，需要用到 `@PrepareForTest` 。 `PowerMockito.mockStatic(Employee.class)` 告诉PowerMock我们需要mock类Employee中所有的静态方法。 `PowerMockito.when(Employee.count()).thenReturn(900)` ，很显然，这句话就是告诉PowerMock当执行 `Employee.count() `时，返回900。很显然，我们需要assert的是 `employeeService.getEmployeeCount()` 的返回值是不是等于900。至此，该测试就算完成了。

但是，如果`EmployeeService`中有一个函数需要调用一个返回值为void的静态方面，我们又该如何处理。比如，`EmployeeService`s中有这样一个方法:

``` java
public boolean giveIncrementToAllEmployeesOf(intpercentage) {
   try{
      Employee.giveIncrementOf(percentage);
      return true;
   } catch(Exception e) {
      return false;
   }
}
```

其实 `Employee.giveIncrementOf(percentage)` 是一个返回值为void的静态方法。

测试代码如下：

``` java
@Test
   public void shouldReturnTrueWhenIncrementOf10PercentageIsGivenSuccessfully() {
      PowerMockito.mockStatic(Employee.class);

      PowerMockito.doNothing().when(Employee.class);
      Employee.giveIncrementOf(10);

      EmployeeService employeeService = new EmployeeService();
      assertTrue(employeeService.giveIncrementToAllEmployeesOf(10));
    }

   @Test
   public void shouldReturnFalseWhenIncrementOf10PercentageIsNotGivenSuccessfully() {
      PowerMockito.mockStatic(Employee.class);

      PowerMockito.doThrow(new IllegalStateException()).when(Employee.class);
      Employee.giveIncrementOf(10);

      EmployeeService employeeService = new EmployeeService();
      assertFalse(employeeService.giveIncrementToAllEmployeesOf(10));
   }
}
```

可以看到该测试中使用了`PowerMockito.doNothing()`和`PowerMockito.doThrow()`。其中`PowerMockito.doNothing().when(Employee.class)`告诉PowerMock当调类`Employee`时啥也不要干，而` PowerMockito.doThrow(new IllegalStateException()).when(Employee.class)`则表示实际调用类Employee时抛一个异常。这样，我们的两个测试就覆盖了`giveIncrementToAllEmployeesOf()`方法的正常和异常两个方面，测试完成。

另外，如果我们使用`PowerMockito.mockStatic(Employee.class)`mock了Employee的所有静态方法，但是我们又想在测试中真实的调用某个其中一个方法，那么就可以使用`PowerMockito.when(...).thenCallRealMethod()`，例如我们想调用count的真实方法，就可以：

```
PowerMockito.when(Employee.count()).thenCallRealMethod();
```

更多关于PowerMock，请参见[官方文档](https://code.google.com/p/powermock/)。例子中的详细代码，请参见[我的github](https://github.com/jiamaoweilie/MockingStaticMothod)。

