# JPAQueryFactory란?
	- SQL문법과 유사하게 복잡한 쿼리를 제작할 수 있게 도와주는 SPRING의 JPA지원 클래스이다.

ex)
``` java
    private RemainDto firstList(DashBoardModel model) {
        return factory
                .select(
                        Projections.fields(
                                RemainDto.class,
                                cfProductInvest.amount.sum().coalesce(0L).as("total"),
                                new CaseBuilder()
                                        .when(cfProduct.category.eq("2"))
                                        .then(cfProductInvest.amount)
                                        .otherwise(0L).sum().coalesce(0L).as("estate"),
                                new CaseBuilder()
                                        .when(cfProduct.category.ne("2"))
                                        .then(cfProductInvest.amount)
                                        .otherwise(0L).sum().coalesce(0L).as("noneEstate")
                        )
                )
                .from(cfProductInvest)
                .leftJoin(cfProduct)
                .on(cfProductInvest.productIdx.eq(cfProduct.idx))
                .leftJoin(g5Member)
                .on(cfProductInvest.memberIdx.eq(g5Member.mbNo))
                .where(
                        g5Member.mbId.eq(model.getMbId()),
                        cfProductInvest.investState.eq("Y")
                )
                .fetchOne();
    }
```


---