---
layout: post
title: "BA Methods Summaries (Part I) - Control variable method"
category: en
tags: Business-Analysis
cover: "http://ww2.sinaimg.cn/large/6d0af205jw1evtvrnmx37j20q10d9n2e.jpg"
---

There was a tough issue about requirement analysising I ever met on one of supply management projects. It was mainly to address BOQ changing case when customers do not need the products any more.

In the perspective of customers, when they don't want some products any more, all they need to do is just deleting these products on the system and then contact the supplyer. However, to supplyers, they must consider it as two different cases.  One case is that if these products have not shipped, they just delete them on the original BOQ list. Another case is that if these products have been shipped, it won't be easy to do. Other than delete them on BOQ list, they must return these shipped products. Therefore, a return BOQ list must be added on system to address this return order.

Through this case, we can see that suppliers are not the same as customers to address BOQ changing. Obviously, this difference must be displayed on IT systems as well. In the PRM system, two BOQ are used to address returning and canceling product cases : 

~~~
MBOQ -- result of BOQ after changing
DBOQ -- changes of BOQ
~~~

For instance, there is a BOQ1 which includes 5 sets of product A with 50 item1 ($3) and 30 item2 ($4).

| BOQ Name | Product Name | Qty of Product | Item Name | Qty of Item | Item Price |
| ------ |:-------- |:-------- |:-------- |:-------- |:-------- |
| BOQ1   | Product A| 5      | item1    | 50     | $3     |
|        |          |          | item2    | 30     | $4      |

Given the information, we can get the amount of product A.

$$
$A =($item1+$item2)*Qty_A=(50*3+30*4)*5
$$

At this moment, customers want to retreat item2 in Product A. So this process goes below:

| BOQ Name | Product Name | Qty of Product | Item Name | Qty of Item | Item Price |
| ------ |:-------- |:-------- |:-------- |:-------- |:-------- |
| MBOQ（After changing）| Product A | 5 | item1 | 50 | $4 |
|                     |       |     | item2 | 0  | $4 |
| DBOQ（changes）  | Product A | 5 | item2 | 30 | $4 |

$$
$A = (50*3+0*4)*5
$$

However, in this case, suppliers must add a new BOQ for products that have been returned while original BOQ can not be changed.

| BOQ Name | Product Name | Qty of Product | Item Name | Qty of Item | Item Price |
| ------ |:-------- |:-------- |:-------- |:-------- |:-------- |
|Orig BOQ1（Not changed）       |Product A |5  |item1 |50 |$3|
|                     |      |     |item2 |30 |$4|
|Return BOQ（similar as DBOQ）|Product A |5  |item2 |30 |$4|

#### How to address this issue for back-end system ?

1. Update MBOQ, then do some tricky on DBOQ
2. Only take DBOQ, then estimate whether delete items in original BOQ or add a new return BOQ.

Finally, we choose solution 1. However, another issue comes.

#### How to identify whether solution 1 adapt for all cases ?

For example, customers may have changed the amount of the rest of products while they cancel products. How to deal with this complex case ? It seems to be hard to list a table include all those cases. But we can try an another method: **Control variable method**. Process is below:

1.**Find all variables**, such as Qty of Product (X), Qty of Item (Y) and Price of item (Z).

2.**Equation and derivation**
  Assuming that only the quantity of item can be changed, others are constant. Then we just need to identify:
  
  $$
  $A_{oBOQ} = $A_{MBOQ}+$A_{DBOQ}
  $$

  $$
  $A_{oBOQ} = $item1+$item2
  $$

  E.g. Only item2 has changed, so item1 can be considerd as a constant Q:
  
  $$
  $A=Q+XYZ
  $$

  $$
  $A_{oBOQ} = $A_{MBOQ}+$A_{DBOQ}=[Q+X*(Y-△y)*Z]+X*△y*Z=Q+X*Y*Z
  $$

  **△y**: Qty of returned item2
  **X**: Qty of Product A
  **Y**: Qty of item2
  **Z**: Price of item2

  Done!

3.**Identification**
  Assuming that there are more than 3 variables have changed, all we need to do is just identifying

  whether

  $$
  $A_{MBOQ}+$A_{DBOQ} = [Q+(X-△x)*Y*Z+△x*(Y-△y)*Z]+△x*△y*Z 
  $$

  is equal to 

  $$
  Q+X*Y*Z
  $$

4.**Find exceptions**
  If some cases failed, we can think about whether these cases exist actually. If not, this solution works.





