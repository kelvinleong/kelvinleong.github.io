---
layout: post
title:  "Hibernate System Exception"
date:   2015-09-15 20:24:58
categories: Java
---

I accidentally encountered the following hibernate system exception after I modified the xml file.

> ***hibernateSystemException Don’t change the reference to a collection with cascade=”all-delete-orphan”***

The explanation told nothing at all.. So I try google or stackOverflow, none of them provided a practical solution for me.

Finally, I examined my xml carefully again and again, then suddenly I realize that one of the bean name is reused again.

After I corrrect it, the exception disappared!

```java
<bean id="tradingAccCredit" class="com.cmms.sharedlib.ControlGroup">
    <property name="controls">
    <util:list value-type="com.cmms.sharedlib.ClientMaint">
            <ref bean="tradingLimitCM" />
            <ref bean="expDtOfTradeLimitCM" />
            <ref bean="tradingLoanLimitCM" /> // The bean name is used previously in another xml
            <ref bean="expDtOfLoanLimitCM" />
            <ref bean="creditRatingCM" />
    </util:list>
    </property>
    <property name="userRoles">
    <util:list value-type="com.cmms.sharedlib.ControlUserRole">
    <ref bean="mo_editMode" />
    <ref bean="mom_editMode" />
    </util:list>
    </property>
</bean>
```

To sum up, make sure the bean names are unique.
