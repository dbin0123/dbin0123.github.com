---
layout: post
title: 2021年05月17日 hibernate-validate使用
date: 2021-05-17
description: "hibernate-validate使用"
tags: hibernate-validate 校验
---   
#### hibernate-validate
##### 什么是hibernate-validate

Hibernate Validator是Hibernate提供的一个开源框架，使用注解方式非常方便的实现服务端的数据校验。

[hibernate-validate官网](http://hibernate.org/validator/)


##### spring boot使用
> spring boot 默认集成了hibernate-validator

###### 引入jar(maven)
```xml
<dependency>
    <groupId>org.hibernate.validator</groupId>
    <artifactId>hibernate-validator</artifactId>
    <version>last-version</version>
</dependency>
```

###### 新增配置类
```java
import org.hibernate.validator.HibernateValidator;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.validation.beanvalidation.MethodValidationPostProcessor;

import javax.validation.Validation;
import javax.validation.Validator;
import javax.validation.ValidatorFactory;

@Configuration
public class ValidatorConfig {

	@Bean
	public MethodValidationPostProcessor methodValidationPostProcessor() {
		MethodValidationPostProcessor postProcessor = new MethodValidationPostProcessor();
		/**设置validator模式为快速失败返回*/
		postProcessor.setValidator(validator());
		return postProcessor;
	}

	@Bean
	public Validator validator() {
		ValidatorFactory validatorFactory = Validation.byProvider(HibernateValidator.class)
				.configure()
				.failFast(true)
				.buildValidatorFactory();
		Validator validator = validatorFactory.getValidator();

		return validator;
	}
}
```

###### 使用示例
```java

import com.fasterxml.jackson.annotation.JsonIgnoreProperties;
import com.aiwiown.constraints.Dict;
import java.io.Serializable;
import java.math.BigDecimal;
import java.util.List;
import javax.validation.constraints.DecimalMax;
import javax.validation.constraints.DecimalMin;
import javax.validation.constraints.Max;
import javax.validation.constraints.Min;
import javax.validation.constraints.NotBlank;
import javax.validation.constraints.NotEmpty;
import javax.validation.constraints.NotNull;
import javax.validation.constraints.Pattern;
import lombok.Data;
import org.hibernate.validator.group.GroupSequenceProvider;

@Data
@JsonIgnoreProperties(ignoreUnknown = true)
public class TestFrom implements Serializable {

    private static final long serialVersionUID = -8635187916891118177L;

    /**数据类型[S(收款)|Q(收入及成本的确认)|F(开票)|R(退货)]*/
    @Dict(value = {"S", "Q", "F", "R"}, message = "数据类型无效[S(收款)|Q(收入及成本的确认)|F(开票)|R(退货)]", hasRequer = true)
    private String dataType;
    /**商户号*/
    @NotBlank(message = "商户号不能为空")
    private String bizId;
    /**子商户号*/
    @NotBlank(message = "子商户号不能为空")
    private String bizSid;
    /**订单编号(T0000000001)*/
    @NotBlank(message = "订单编号不能为空")
    private String tradeNo;
    @NotBlank(message = "支付单号不能为空")
    private String paymentNo;
    @Dict(value = {"微信", "支付宝", "网银", "其他"}, message = "收款方式无效[微信/支付宝/网银/其他]", groups = {TestValidate.Group1Check.class, TestValidate.Group4Check.class})
    private String paymentMethod;
    @NotEmpty(message = "销售渠道不能为空", groups = {TestValidate.Group1Check.class})
    private String salesPlatform;
    private String shopName;
    @NotNull(message = "手续费不能为空", groups = {TestValidate.Group1Check.class})
    @DecimalMin(value = "0.00", message = "手续费必须大于等于0", groups = {TestValidate.Group1Check.class, TestValidate.Group4Check.class})
    private BigDecimal serviceFee;
    @NotNull(message = "运费不能为空", groups = {TestValidate.Group1Check.class})
    @DecimalMin(value = "0.00", message = "运费必须大于等于0", groups = {TestValidate.Group1Check.class, TestValidate.Group4Check.class})
    private BigDecimal freightFee;
    private BigDecimal discountFee;
    private String discountType;
    @NotNull(message = "支付时间不能为空", groups = {TestValidate.Group1Check.class, TestValidate.Group4Check.class})
    @Pattern(regexp = "^[1-9]\\d{3}-(0[1-9]|1[0-2])-(0[1-9]|[1-2][0-9]|3[0-1])\\s+(20|21|22|23|[0-1]\\d):[0-5]\\d:[0-5]$", message = "支付时间格式不正确[yyyy-MM-dd HH:mm:ss]")
    private String paymentTime;
    @NotNull(message = "收货时间不能为空", groups = {TestValidate.Group2Check.class})
    @Pattern(regexp = "^[1-9]\\d{3}-(0[1-9]|1[0-2])-(0[1-9]|[1-2][0-9]|3[0-1])\\s+(20|21|22|23|[0-1]\\d):[0-5]\\d:[0-5]$", message = "收货时间格式不正确[yyyy-MM-dd HH:mm:ss]")
    private String receiptTime;
    @NotEmpty(message = "订单商品详情不能为空")
    private List<TradeOrder> listOrder;

    @Data
    public static class TradeOrder implements Serializable {
        private static final long serialVersionUID = -9141355495258606857L;
        /**订单交易明细单号(OT00000000011)*/
        @NotEmpty(message = "订单交易明细单号不能为空")
        private String orderNo;
        @Dict(value = {"自营", "第三方入驻"}, message = "商品类型无效(自营/第三方入驻)", hasRequer = true, groups = {TestValidate.Group1Check.class})
        private String orderItemChannel;
        @Dict(value = {"主要责任人", "代理人"}, message = "履约义务类型无效(主要责任人/代理人)", hasRequer = true, groups = {TestValidate.Group1Check.class})
        private String obligationType;
        @NotEmpty(message = "商品ID不能为空", groups = {TestValidate.Group1Check.class})
        private String orderItemId;
        @NotEmpty(message = "商品名称不能为空", groups = {TestValidate.Group1Check.class})
        private String orderItemName;
        @NotEmpty(message = "商品类别不能为空", groups = {TestValidate.Group1Check.class})
        private String orderItemType;
        @NotEmpty(message = "详细信息不能为空", groups = {TestValidate.Group1Check.class})
        private String orderItemDesc;
        @NotNull(message = "销售数量不能为空", groups = {TestValidate.Group1Check.class, TestValidate.Group4Check.class})
        @Min(value = 0, message = "付款销售数量必须大于等于0", groups = {TestValidate.Group1Check.class})
        @Max(value = 0, message = "退款销售数量必须小于等于0", groups = {TestValidate.Group4Check.class})
        private Integer salesVolume;
        @NotNull(message = "销售单价不能为空", groups = {TestValidate.Group1Check.class, TestValidate.Group2Check.class, TestValidate.Group4Check.class})
        @DecimalMin(value = "0.00", message = "销售单价必须大于等于0", groups = {TestValidate.Group1Check.class, TestValidate.Group2Check.class, TestValidate.Group4Check.class})
        private BigDecimal salesUnitPrice;
        @NotNull(message = "销售金额不能为空", groups = {TestValidate.Group1Check.class, TestValidate.Group2Check.class, TestValidate.Group4Check.class})
        @DecimalMin(value = "0.00", message = "付款销售金额必须大于等于0", groups = {TestValidate.Group1Check.class, TestValidate.Group2Check.class})
        @DecimalMax(value = "0.00", message = "退款销售金额必须小于等于0", groups = {TestValidate.Group4Check.class})
        private BigDecimal salesAmount;
        @NotNull(message = "销售税率不能为空", groups = {TestValidate.Group1Check.class, TestValidate.Group2Check.class, TestValidate.Group4Check.class})
        @DecimalMin(value = "0.00", message = "销售税率必须大于等于0", groups = {TestValidate.Group1Check.class, TestValidate.Group2Check.class, TestValidate.Group4Check.class})
        private BigDecimal salesTaxRate;
        @NotNull(message = "实收金额不能为空", groups = {TestValidate.Group1Check.class, TestValidate.Group4Check.class})
        @DecimalMin(value = "0.00", message = "付款实收金额必须大于等于0", groups = {TestValidate.Group1Check.class})
        @DecimalMax(value = "0.00", message = "退款实收金额必须小于等于0", groups = {TestValidate.Group4Check.class})
        private BigDecimal actualAmount;
        @NotNull(message = "收入金额不能为空", groups = {TestValidate.Group2Check.class})
        @DecimalMin(value = "0.00", message = "收入金额必须大于等于0", groups = {TestValidate.Group2Check.class})
        private BigDecimal incomeAmount;
        @NotNull(message = "结算单价不能为空", groups = {TestValidate.Group2Check.class})
        @DecimalMin(value = "0.00", message = "结算单价必须大于等于0", groups = {TestValidate.Group2Check.class})
        private BigDecimal settlementUnitPrice;
        @NotNull(message = "结算数量不能为空", groups = {TestValidate.Group2Check.class})
        @Min(value = 0, message = "结算数量必须大于等于0", groups = {TestValidate.Group2Check.class})
        private Integer settlementVolume;
        @NotNull(message = "结算金额不能为空", groups = {TestValidate.Group2Check.class})
        @DecimalMin(value = "0.00", message = "结算金额必须大于等于0", groups = {TestValidate.Group2Check.class})
        private BigDecimal settlementAmount;
        @NotNull(message = "采购税率不能为空", groups = {TestValidate.Group2Check.class})
        @DecimalMin(value = "0.00", message = "采购税率必须大于等于0", groups = {TestValidate.Group2Check.class})
        private BigDecimal purchaseTaxRate;
        @NotNull(message = "成本金额不能为空", groups = {TestValidate.Group2Check.class})
        @DecimalMin(value = "0.00", message = "成本金额必须大于等于0", groups = {TestValidate.Group2Check.class})
        private BigDecimal costAmount;
        @NotNull(message = "结存数量不能为空")
        @Min(value = 1, message = "结存数量必须大于1")
        private Integer balanceQuantity;
        @NotNull(message = "开票单价不能为空", groups = {TestValidate.Group3Check.class, TestValidate.Group4Check.class})
        @DecimalMin(value = "0.00", message = "开票单价必须大于等于0", groups = {TestValidate.Group3Check.class})
        @DecimalMax(value = "0.00", message = "开票单价必须小于等于0", groups = {TestValidate.Group4Check.class})
        private BigDecimal invoiceUnitPrice;
        @NotNull(message = "开票数量不能为空", groups = {TestValidate.Group3Check.class, TestValidate.Group4Check.class})
        @Min(value = 0, message = "开票数量必须大于等于0", groups = {TestValidate.Group3Check.class, TestValidate.Group4Check.class})
        private Integer invoicedQuantity;
        @NotNull(message = "开票金额不能为空", groups = {TestValidate.Group3Check.class, TestValidate.Group4Check.class})
        @DecimalMin(value = "0.00", message = "开票金额必须大于等于0", groups = {TestValidate.Group3Check.class})
        @DecimalMax(value = "0.00", message = "开票金额必须小于等于0", groups = {TestValidate.Group4Check.class})
        private BigDecimal invoiceAmount;
        @NotNull(message = "开票税率不能为空", groups = {TestValidate.Group3Check.class, TestValidate.Group4Check.class})
        @DecimalMin(value = "0.00", message = "开票税率必须大于等于0", groups = {TestValidate.Group3Check.class, TestValidate.Group4Check.class})
        private BigDecimal invoiceBillRate;
        @NotNull(message = "开票日期不能为空", groups = {TestValidate.Group3Check.class})
        @Pattern(regexp = "^[1-9]\\d{3}-(0[1-9]|1[0-2])-(0[1-9]|[1-2][0-9]|3[0-1])$", message = "开票日期格式不正确[yyyy-MM-dd]", groups = {TestValidate.Group3Check.class})
        private String invoiceOpenDate;
        @NotNull(message = "退货日期不能为空", groups = {TestValidate.Group4Check.class})
        @Pattern(regexp = "^[1-9]\\d{3}-(0[1-9]|1[0-2])-(0[1-9]|[1-2][0-9]|3[0-1])$", message = "退货日期格式不正确[yyyy-MM-dd]", groups = {TestValidate.Group3Check.class})
        private String refundTime;
        private List<OrderDiscount> listDiscount;

        @Data
        public static class OrderDiscount implements Serializable {
            @NotNull(message = "商品折扣金额不能为空", groups = {TestValidate.Group1Check.class, TestValidate.Group2Check.class, TestValidate.Group4Check.class})
            @DecimalMin(value = "0.00", message = "商品折扣金额必须大于等于0", groups = {TestValidate.Group1Check.class, TestValidate.Group2Check.class, TestValidate.Group4Check.class})
            private BigDecimal discountFee;
            @NotBlank(message = "商品折扣类型不能为空", groups = {TestValidate.Group1Check.class, TestValidate.Group2Check.class, TestValidate.Group4Check.class})
            private String discountType;
        }
    }
}

```
###### 分组校验使用

- 新增分组校验类A实现DefaultGroupSequenceProvider接口
```java
public class TestValidate implements DefaultGroupSequenceProvider<TestFrom> {}
```
- 实现getValidationGroups方法
```java
    @Override
    public List<Class<?>> getValidationGroups(TestFrom testFrom) {return null;}
```
- 按需求新增分组校验逻辑

```java

    @Override
    public List<Class<?>> getValidationGroups(TestFrom testFrom) {
        List<Class<?>> listGroudSequence = new ArrayList<>();
        listGroudSequence.add(TestFrom.class);
        if (Objects.nonNull(financialStatementFrom)) {
            //[S(group1)|Q(group2)|F(group3)|R(group4)]
            String dataType = testFrom.getDataType();
            switch (dataType) {
                case DATA_TYPE_GROUP1:
                    listGroudSequence.add(TestValidate.Group1Check.class);
                    break;
                case DATA_TYPE_GROUP2:
                    listGroudSequence.add(TestValidate.Group2Check.class);
                    break;
                case DATA_TYPE_GROUP3:
                    listGroudSequence.add(TestValidate.Group3Check.class);
                    break;
                case DATA_TYPE_GROUP4:
                    listGroudSequence.add(TestValidate.Group4Check.class);
                    break;
            }
        }
        return listGroudSequence;
    }

    public interface Group1Check {
    }

    public interface Group2Check {
    }

    public interface Group3Check {
    }

    public interface Group4Check {
    }

```

###### spring boot使用

```java 

    @RequestMapping("/test")
    public JSONResult test(@RequestBody TestFrom testFrom, BindingResult bindingResult) {
        log.info("入参:{}", testFrom);
        //参数校验 Default.class用于全局参数校验
        ValidatorUtils.validateFastException(testFrom, {TestValidate.Group1Check.class,Default.class});
        //TODO ... 业务逻辑
        return JSONResult.ok();
    }
    
```


控制层类




#### 附录
##### 相关注解
<table>
	<tr>
	    <th>分类</th>
	    <th>注解</th>
	    <th>描述</th>  
	</tr >
	<tr >
	    <td rowspan="4">空和非空检查</td>
	    <td>@Null</td>
	    <td>被注释的元素必须为null</td>
	</tr>
	<tr>
	    <td>@NotNull</td>
	    <td>被注释的元素必须不为null</td>
	</tr>
	<tr>
	    <td>@NotEmpty</td>
	    <td>被注释的元素的必须非空(字符串长度不为0,集合长度不为0)</td>
	</tr>
	<tr>
	    <td>@NotBlank</td>
	    <td>注释元素的值不为空(不为null, 去首尾空格后长度为0</td>
	</tr>
	<tr >
	    <td rowspan="2">Boolean检查</td>
	    <td>@AssertFalse</td>
	    <td>元素值必须为false</td>
	</tr>
	<tr>
	    <td>@AssertTrue</td>
	    <td>元素值必须为true</td>
	</tr>
	<tr>
	    <td rowspan="1">长度检查</td>
	    <td>@Size(max, min)</td>
	    <td>元素值(字符串)的长度需在min,max之间(between and)</td>
	</tr>
	<tr >
	    <td rowspan="5">日期时间检查</td>
	    <td>@Past</td>
	    <td>元素必须为过去时间</td>
	</tr>
	<tr>
	    <td >@Future</td>
	    <td>元素的值必须为当前时间之后</td>
	</tr>
	<tr>
	    <td >@FutureOrPresent</td>
	    <td>元素的值必须为当前时间或当前时间之后</td>
	</tr>
	<tr>
	    <td >@Past</td>
	    <td>元素的值必须为当前时间之前</td>
	</tr>
	<tr>
	    <td >@PastOrPresent</td>
	    <td>元素的值必须为当前时间或当前时间之前</td>
	</tr>
	<tr>
	    <td  rowspan="9">数值校验</td>
	    <td >@Max(value)</td>
	    <td >元素的值必须大于等于指定值</td>
	</tr>
	<tr>
	    <td >@Min(value)</td>
	    <td >元素的值必须小于等于指定值</td>
	</tr>
		<tr>
	    <td >@DecimalMax(value)</td>
	    <td >元素的值必须大于等于指定值</td>
	</tr>
		<tr>
	    <td >@DecimalMin(value)</td>
	    <td >元素的值必须小于等于指定值</td>
	</tr>
	<tr>
	    <td >@Digits(integer, function)</td>
	    <td >元素值必须是一个数字，其值必须在可接受的范围内</td>
	</tr>
	<tr>
	    <td >@Negative</td>
	    <td >元素值必须为负数</td>
	</tr>
	<tr>
	    <td >@NegativeOrZero</td>
	    <td >元素值必须为负数或0</td>
	</tr>
	<tr>
	    <td >@Positive</td>
	    <td >元素值必须为这正整数</td>
	</tr>
	<tr>
	    <td >@PositiveOrZero</td>
	    <td >元素值必须为这正整数或0</td>
	</tr>
	<tr>
	    <td rowspan="1">正则校验</td>
	    <td>@Pattern(value)</td>
	    <td>元素值必须符合正则表达式</td>
	</tr>
	<tr>
	    <td rowspan="1">邮箱校验</td>
	    <td>@Email</td>
	    <td>元素值必须为邮箱</td>
	</tr>
	<tr>
	    <td rowspan="1">URL校验</td>
	    <td>@URL(protocol=,host=, port=,regexp=, flags=)</td>
	    <td>元素值为URL</td>
	</tr>
	<tr>
	    <td rowspan="1">信用卡校验</td>
	    <td>@CreditCardNumber</td>
	    <td>元素值为必须必须通过Luhn校验算法</td>
	</tr>
</table>


##### 异常处理
```java
@Slf4j
@ControllerAdvice
public class HibernateValidatedExceptionHandler {
 
    /**
     * 处理@Validated参数校验失败异常
     * @param exception 异常类
     * @return 响应
     */
    @ResponseBody
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ModelAndView exceptionHandler(MethodArgumentNotValidException exception){
        BindingResult bindingResult = exception.getBindingResult();
        StringBuilder stringBuilder = new StringBuilder();
        FieldError fieldError = bindingResult.getFieldError();
		MappingJackson2JsonView view = new MappingJackson2JsonView();
		view.setExtractValueFromSingleKeyModel(true);
		ModelAndView modelAndView = new ModelAndView(view);
		modelAndView.addObject(fieldError);
        return modelAndView;
    }
}
```

##### 级联校验工具类型
```java

import com.aiwiown.exceptions.CommonException;
import java.util.Set;
import javax.validation.ConstraintViolation;
import javax.validation.Validation;
import javax.validation.Validator;
import org.hibernate.validator.HibernateValidator;

public class ValidatorUtils {

    static Validator validatorFast = Validation.byProvider(HibernateValidator.class).configure().failFast(true).buildValidatorFactory().getValidator();
    static Validator validatorAll = Validation.byProvider(HibernateValidator.class).configure().failFast(false).buildValidatorFactory().getValidator();

    /**
     * 校验遇到第一个不合法的字段直接返回不合法字段，后续字段不再校验
     * @param domain 校验对象
     * @param <T>
     * @return
     * @throws Exception
     */
    public static <T> Set<ConstraintViolation<T>> validateFast(T domain, Class<?>... groups) {
        return validatorFast.validate(domain, groups);
    }

    /**
     * 校验遇到第一个不合法的字段直接返回不合法字段，后续字段不再校验(存在不合法参数 异常提示)
     * @param domain 校验对象
     * @param <T>
     * @return
     * @throws Exception
     */
    public static <T> void validateFastException(T domain, Class<?>... groups) {
        Set<ConstraintViolation<T>> constraintViolations = validateFast(domain, groups);
        if (null != constraintViolations && constraintViolations.size() > 0) {
            ConstraintViolation<T> violation = constraintViolations.iterator().next();
            throw new CommonException(violation.getMessage());
        }
    }


    /**
     * 校验所有字段并返回不合法字段
     * @param domain
     * @param <T>
     * @return
     * @throws Exception
     */
    public static <T> Set<ConstraintViolation<T>> validateAll(T domain, Class<?>... groups) {
        return validatorAll.validate(domain, groups);
    }


    /**
     * 校验所有字段 存在不合法的异常
     * @param domain 校验对象
     * @param <T>
     * @throws Exception
     */
    public static <T> void validateAllException(T domain, Class<?>... groups) {
        Set<ConstraintViolation<T>> constraintViolations = validateAll(domain, groups);
        if (null != constraintViolations && constraintViolations.size() > 0) {
            ConstraintViolation<T> violation = constraintViolations.iterator().next();
            throw new CommonException(violation.getMessage());
        }
    }
}
```
