---
layout: post
title: 2021年03月20日 稀疏数组
date: 2021-03-20
description: "稀疏数组"
tags: 数组 稀疏数组 数据压缩
---   
#### 稀疏数组
##### 什么是稀疏数组?
矩阵(二维数组)中(非零/非空/不相似值)元素的个数远远小于矩阵元素的总数，并且(非零/非空/不相似值)元素的分布没有规律，通常认为矩阵中(非零/非空/不相似值)元素的总数比上矩阵所有元素总数的值小于等于0.05时，则称该矩阵为稀疏矩阵(sparse matrix)，该比值称为这个矩阵的稠密度;

##### 稀疏数组数据结构
1. 第一行记录数组行数、列数和有多少个值。
2. 把具有不同值得元素依次记录在后面，依次记录行号，列号，值。


##### 示例代码
```java

import java.lang.reflect.Array;
import java.util.Objects;

/**
 * @ClassName : SparseArray
 * @Description : 稀疏数组
 * @Author : dbin0123
 * @Date: 2021-03-20 20:03
 */
public class SparseArray {


    /**
     * 二维数组转稀疏数组(转换)
     * 要求对象必须实现hashcode equals方法
     *
     * @param array
     * @return
     */
    public static <T extends Object> Object[][] toSparseArray(T[][] array, T commonalityData) {
        if (null == array || array.length == 0) {
            throw new RuntimeException("原数组对象/数据不能为空");
        }
        int sparseArrayRowNum = 0;
        for (Object[] dataRow : array) {
            for (Object data : dataRow) {
                if (!Objects.equals(data, commonalityData)) {
                    sparseArrayRowNum++;
                }
            }
        }
        Object[][] arrayObject = new Object[sparseArrayRowNum + 1][3];
        arrayObject[0][0] = array.length;
        arrayObject[0][1] = array[0].length;
        arrayObject[0][2] = sparseArrayRowNum;
        //保存数据至稀疏数组中
        int indxe = 0;
        for (int i = 0; i < array.length; i++) {
            for (int i1 = 0; i1 < array[i].length; i1++) {
                if (!Objects.equals(array[i][i1], commonalityData)) {
                    indxe++;
                    arrayObject[indxe][0] = i;
                    arrayObject[indxe][1] = i1;
                    arrayObject[indxe][2] = array[i][i1];
                }
            }
        }
        return arrayObject;
    }

    /**
     * 稀疏数组转二维数组(还原)
     *
     * @param sparseArray
     * @param fullData
     * @param clazz
     * @param <T>
     * @return
     */
    public static <T extends Object> T[][] restoreArray(Object[][] sparseArray, T fullData, Class<T> clazz) {
        if (null == sparseArray || sparseArray.length < 1) {
            throw new RuntimeException("sparseArray不能为空");
        }
        T[][] arr = (T[][]) Array.newInstance(clazz, (Integer) sparseArray[0][0], (Integer) sparseArray[0][1]);
        //填充数据
        for (int i = 0; i < arr.length; i++) {
            for (int i1 = 0; i1 < arr[i].length; i1++) {
                arr[i][i1] = fullData;
            }
        }
        for (int i = 1; i < sparseArray.length; i++) {
            arr[(Integer) sparseArray[i][0]][(Integer) sparseArray[i][1]] = (T) sparseArray[i][2];
        }
        return arr;
    }

    public static void main(String[] args) {
        Persion[][] arr = new Persion[10][10];
        arr[1][2] = new Persion("1", 10);
        arr[2][2] = new Persion("2", 10);
        arr[2][3] = new Persion("1", 10);
        arr[3][3] = new Persion("2", 10);
        System.out.println("原数据:");
        for (int i = 0; i < arr.length; i++) {
            for (int j = 0; j < arr[i].length; j++) {
                System.out.print(arr[i][j] + "\t");
            }
            System.out.println();
        }
        System.out.println("稀疏数组:");
        Object[][] objects = SparseArray.toSparseArray(arr, null);
        for (int i = 0; i < objects.length; i++) {
            for (int j = 0; j < objects[i].length; j++) {
                System.out.print(objects[i][j] + "\t");
            }
            System.out.println();
        }
        System.out.println("还原数据:");
        Persion[][] objects1 = SparseArray.restoreArray(objects, null, Persion.class);

        for (int i = 0; i < objects1.length; i++) {
            for (int j = 0; j < objects1[i].length; j++) {
                System.out.print(objects1[i][j] + "\t");
            }
            System.out.println();
        }
    }

    public static class Persion {
        String name;
        Integer age;

        public Persion() {
        }

        public Persion(String name, Integer age) {
            this.name = name;
            this.age = age;
        }

        @Override
        public String toString() {
            return "{" +
                    "name:'" + name + '\'' +
                    ", age:" + age +
                    '}';
        }

        @Override
        public boolean equals(Object o) {
            if (this == o) {
                return true;
            }
            if (o == null || getClass() != o.getClass()) {
                return false;
            }
            Persion persion = (Persion) o;
            return Objects.equals(name, persion.name) && Objects.equals(age, persion.age);
        }

        @Override
        public int hashCode() {
            return Objects.hash(name, age);
        }
    }
}
```

> 运行结果

```text
原数据:
null	null	null	null	null	null	null	null	null	null	
null	null	{name:'1', age:10}	null	null	null	null	null	null	null	
null	null	{name:'2', age:10}	{name:'1', age:10}	null	null	null	null	null	null	
null	null	null	{name:'2', age:10}	null	null	null	null	null	null	
null	null	null	null	null	null	null	null	null	null	
null	null	null	null	null	null	null	null	null	null	
null	null	null	null	null	null	null	null	null	null	
null	null	null	null	null	null	null	null	null	null	
null	null	null	null	null	null	null	null	null	null	
null	null	null	null	null	null	null	null	null	null	
稀疏数组:
10	10	4	
1	2	{name:'1', age:10}	
2	2	{name:'2', age:10}	
2	3	{name:'1', age:10}	
3	3	{name:'2', age:10}	
还原数据:
null	null	null	null	null	null	null	null	null	null	
null	null	{name:'1', age:10}	null	null	null	null	null	null	null	
null	null	{name:'2', age:10}	{name:'1', age:10}	null	null	null	null	null	null	
null	null	null	{name:'2', age:10}	null	null	null	null	null	null	
null	null	null	null	null	null	null	null	null	null	
null	null	null	null	null	null	null	null	null	null	
null	null	null	null	null	null	null	null	null	null	
null	null	null	null	null	null	null	null	null	null	
null	null	null	null	null	null	null	null	null	null	
null	null	null	null	null	null	null	null	null	null	
```