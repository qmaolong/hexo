---
title: 从业务代码看Kotlin
date: 2019-05-09 14:22:00
tags:
categories:
---

Kotlin的名声想必所有Java开发者都有所耳闻，其方便简洁的语法使得在业务开发中省去不少事情，也增加了代码的美观度。

对于Kotlin的教程不多做赘述，传送门奉上https://www.kotlincn.net/docs/reference/。

先上一段代码

```
fun foods(deskName: String, model: Model, @PathVariable("shopIdStr") shopIdStr: String, request: HttpServletRequest): String {
        val shop = shopService.findByObjectId(ObjectId(shopIdStr)) ?: return msgPage("error", "未知门店", model)
        //验证桌台可用性
        var desk: Desk? = null
        shop.rooms?.filter { it.desks != null }?.forEach {
            it.desks?.filter { it.name == deskName }?.forEach { desk = it }
        }
        desk ?: return msgPage("error", "桌台号【$deskName】不存在", model)
        //获取类别
        val smallCats = ArrayList<SmallCategory>()
        shop.categories?.forEach {
            smallCats.addAll(it.children)
        }

        val moConfig = shopService.findMoConfig(shop.objectId!!)
        //快餐模式时获取活动
        val specialRules = if (moConfig.mode == Mode.Fast)saleService.findSpecialRuleByShopId(shop.objectId!!) else arrayListOf()
        //获取菜品
        val foods = foodService.generateMoFoods(shop.objectId!!, specialRules, moConfig)?: arrayListOf()

        for (cate in smallCats) {
            cate.foods!!.addAll(foods.filter { it.cat == cate.id })
        }
        model.addAttribute("cats", JSONObject.toJSON(smallCats))
        model.addAttribute("deskName", deskName)
        model.addAttribute("shopName", shop.name)
        model.addAttribute("specialRules", JSONObject.toJSON(specialRules))
        val moUser = MoUserUtil.getUserFromAttributeOrCookie(request)
        model.addAttribute("lastValidOrderId", orderService.findLastValidOrderId(moUser!!.id, shop.objectId!!))
        return "foods-for-self-order"
    }
```