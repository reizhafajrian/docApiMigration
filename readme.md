## MIGRATION API IMPORTIR.ID FROM LARAVEL TO NODE.JS

[![N|Solid](https://is2-ssl.mzstatic.com/image/thumb/Purple125/v4/e9/c2/ac/e9c2ac4b-0405-684c-3307-f84f44815687/AppIcon-0-0-1x_U007emarketing-0-0-0-5-0-0-sRGB-0-0-0-GLES2_U002c0-512MB-85-220-0-0.png/460x0w.png)]()

Migrasi a few rest api fron laravel to node.js.

## Function
**findProductDetailOrder()**

the function is have same result which is get YgProductDetail table from database

in Laravel:
```

    public function flash_times(){
        return $this->hasMany('App\Yiwugo\FlashSaleTime', 'yg_product_id','product_id')->orderBy('start','asc');
    }//this relation function from YgProductDetail.php to get flash time table
public function getYgProductDetailByProductId($productId)
    {
        $query  = YgProductDetail::with(['flash_time','categories.category'])
            ->where('product_id', strval($productId));
        return $query->first();
    }
```
in node.js:
```
const FlashTime = database.model("FlashTime", {
  hasTimestamps: true,
  tableName: "flash_sale_time",
  idAttribute: "id",
});

const YgProductDetail = database.model("ProductDetail", {
  hasTimestamps: true,
  tableName: "yg_product_detail",
  idAttribute: "id",
  flash_time: function() {
    return this.belongsTo(FlashTime, "product_id", "yg_product_id").orderBy(
      "id",
      "desc"
    );
  },
  categories: function() {
    return this.hasMany(YgProductCategory, "product_id");
  },
});
export const findProductDetailOrder = (productid) => {
  return new Promise((resolve, reject) => {
    YgProductDetail.where({ product_id: String(productid) })
      .fetch({ withRelated: ["flash_time", "categories.category"] })
      .then((result) => {
        result ? resolve(result.toJSON()) : resolve(null);
      })
      .catch((err) => {
        reject(err);
      });
  });
};
```
**getAllYgOrderDetailByProdId()**

get allYgOrderDetail table with relation with YgOrder Schema and hav a realtion with yg_status

in laravel:
```
    public function statusPaid(){
        return $this->hasOne('App\Yiwugo\YgStatus', 'yg_order_id')->where('title', 'customer paid');
    }//this relation function from YgOrder.php
     public function yg_order()
    {
        return $this->belongsTo('App\Yiwugo\YgOrder');
    }//this relation function from YgOrderDetail.php

    public function getAllYgOrderDetailByProdId($productId)
    {
        $query  = YgOrderDetail::with(['yg_order.statusPaid'])
            ->where('yg_product_id', $productId);
        return $query->get();
    }
```


in node.js:
```
const YgOrderDetail = database.model("OrderDetail", {
  defaults: {
    warehouse_delivery_fee_idr: 0,
    price_per_type: "",
    admin_note: "",
    address: "",
    tracking_note: "",
  },
  hasTimestamps: true,
  tableName: "yg_order_detail",
  idAttribute: "id",
  yg_product_detail: function() {
    return this.belongsTo(YgProductDetail, "yg_product_id", "product_id");
  },
  yg_order: function() {
    return this.belongsTo(YgOrder);
  },
});
const YgOrder = database.model("Order", {
  hasTimestamps: true,
  tableName: "yg_order",
  idAttribute: "id",
  yg_order_detail: function() {
    return this.hasMany(YgOrderDetail, "yg_order_id");
  },
  statusPaid: function() {
    return this.hasOne(YgOrderStatus, "yg_order_id").where(
      "title",
      "customer paid"
    );
  },
});
const YgOrderStatus = database.model("OrderStatus", {
  hasTimestamps: true,
  tableName: "yg_status",
  idAttribute: "id",
});
export const getAllYgOrderDetailByProdId = (prodId) => {
  return new Promise((resolve, reject) => {
    YgOrderDetail.where({ yg_product_id: String(prodId) })
      .fetchAll({ withRelated: ["yg_order.statusPaid"] })
      .then((result) => {
        result ? resolve(result.toJSON()) : resolve(null);
      })
      .catch((err) => {
        reject(err);
      });
  });
};
```
