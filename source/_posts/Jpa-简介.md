---
title: Spring Data Jpa(一)
date: 2016-07-09 13:28:53
tags: Spring Data Jpa
---

### Spring Jpa Date 基础用法
>. = = 首先得有Spring和Hibernate环境,不再赘述.

#### 第一步
 创建一个实体类 MsgMeassage,IdEntity类里面的CreateTime和Id
``` java
@Entity
@Table(name = "msg_meassage")
@DynamicInsert(true)
@DynamicUpdate(true)
public class MsgMeassage extends IdEntity {
  private Doctor doctor;
  private String info;
  public MsgMeassage() {
  }
  @ManyToOne(cascade = CascadeType.REFRESH, optional = true)
  @JoinColumn(name = "doctor_id")
  public Doctor getDoctor() {
      return this.doctor;
  }
  public void setDoctor(Doctor doctor) {
      this.doctor = doctor;
  }
  @Column(name = "info")
  public String getInfo() {
      return info;
  }
  public void setInfo(String info) {
      this.info = info;
  }
}
```

#### 第二步
创建MsgMeassageDao,继承JpaRepository<T,ID类型>和JpaSpecificationExecutor<T>
``` java
public interface MsgMeassageDao extends JpaRepository<MsgMeassage, Long>, JpaSpecificationExecutor<MsgMeassage> {

}
```
#### 第三步
创建MsgMeassageServiceImpl,然后自动注入 MsgMeassageDao ,接下来我们要查询数据的时候就
可以这样写了
***** 比如我想查询所有的MsgMeassage,因为JpaRepository封装了很多常用的方法,我们可以直
接调用 findAll,findOnd,delete,save 等等常用的方法,列子如下:
``` java
    @Resource
    private MsgMeassageDao msgMeassageDao;
    //直接保存 MsgMeassage 对象
    public void save(MsgMeassage msgMeassage) {
        msgMeassageDao.save(msgMeassage);
    }
```

##### 比如我想加入条件查询,比如 查询某一个医生的MsgMeassage就可以这样:
###### SQL代码如下
``` sql
  select * from MsgMeassage ms where ms.doctor=XXX
```
###### Java 代码这样写 达到同样的功能
``` java
  @Resource
  private MsgMeassageDao msgMeassageDao;
  public MsgMeassage getMeassageByDoctor(final Doctor doctor) {
        return msgMeassageDao.findOne(new Specification<MsgMeassage>() {
            @Override
            public Predicate toPredicate(Root<MsgMeassage> root, CriteriaQuery<?> query, CriteriaBuilder cb) {
                List<Predicate> predicateList = Lists.newArrayList();
                predicateList.add(cb.equal(root.get(MsgMeassage_.doctor), doctor));
                return cb.and(predicateList.toArray(new Predicate[predicateList.size()]));
            }
        });
    }
```
######  = = 并且连删除方法都可以这样做（反面教材，很影响性能，千万别这么干）
``` java
  public void delMeassageByDoctor(final Doctor doctor) {
        msgMeassageDao.delete(msgMeassageDao.findAll(new Specification<MsgMeassage>() {
            @Override
            public Predicate toPredicate(Root<MsgMeassage> root, CriteriaQuery<?> query, CriteriaBuilder cb) {
                List<Predicate> predicateList = Lists.newArrayList();
                predicateList.add(cb.equal(root.get(MsgMeassage_.doctor), doctor));
                return cb.and(predicateList.toArray(new Predicate[predicateList.size()]));
            }
        }));
    }
```
###### 本文暂时聊到这里 下一次更新 会详细说明下 怎么使用 Specification 来 用面向对象
的方式来替换sql.

> = = 2333
