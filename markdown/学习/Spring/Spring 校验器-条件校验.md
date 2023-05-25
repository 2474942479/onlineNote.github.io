# Spring 校验器

## 分组校验（不同条件下进行不同的校验）

1.  @Validated 注释controller请求参数

2. 创建分组

   ```java
   public interface WhQuotationDetailGroup {
   
       interface ChargeInfoGroup extends Default{
   
       }
   
       interface ConsumablesRuleGroup extends Default {
       }
   
   
       interface CustomsClearRuleGroup extends Default {
       }
   
       interface OrderNumRuleGroup extends Default {
       }
   
       interface OverOrderRuleGroup extends Default {
       }
   
       interface OrderNumAndConditionGroup extends Default {
   
       }
   
       interface OrderNumOrConditionGroup extends Default {
   
       }
   
   }
   ```

3. 添加校验注解 并分组   存在嵌套对象  添加@Valid

   ```java
   @NotNull(message = "收费类型信息-单价不能为空!", groups = {ChargeInfoGroup.class})
   private BigDecimal price;
   ```

4. 在 业务逻辑中 不同条件下进行校验

   ```java
   BeanValidUtil.validateBean(weightRange, WhQuotationDetailGroup.OrderNumOrConditionGroup.class);
   ```