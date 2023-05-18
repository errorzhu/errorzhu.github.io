# flask sqlalchemy 数据库字段不更新的问题

## 背景
写了个定时任务,周期检查设备状态,发现并不能更新数据库对应的状态,代码大致逻辑如下

```
    def change_status(self, equipment_id):
        try:
            equipment = Equipment.query.filter(
                Equipment.id == int(equipment_id)
            ).first()
            attr = equipment.attr
            attr["serial"]["connected"] = True
            equipment.attr = attr
            db.session.commit()

        except Exception as e:
            logger.exception(e)

```

```
def get_connected_device():
    try:
        with scheduler.app.app_context():
            equipments = Equipment.query.filter(
                and_(
                    Equipment.type == "6",
                    func.json_extract(Equipment.attr, "$.serial.connected") == False,
                )
            ).all()
            for equipment in equipments:
                for device in device_list:
                    if device.check(equipment):
                        # 更新该条记录的 attr属性,这里attr的类型是JSON
                        device.change_status(equipment.id)
                        break
    except Exception as e:
        logger.exception(e)

```

## 原理

查询文档https://docs.sqlalchemy.org/en/13/core/type_basics.html#sqlalchemy.types.JSON,
```
The JSON type, when used with the SQLAlchemy ORM, does not detect in-place mutations to the structure. In order to detect these, the sqlalchemy.ext.mutable extension must be used. This extension will allow “in-place” changes to the datastructure to produce events which will be detected by the unit of work. See the example at HSTORE for a simple example involving a dictionary
```
发现json类型默认是不可变类型

将代码修改如下
```

    def change_status(self, equipment_id):
        try:
            equipment = Equipment.query.filter(
                Equipment.id == int(equipment_id)
            ).first()
            attr = equipment.attr
            attr["serial"]["connected"] = True
            equipment.attr = attr
            # 为orm提供一个事件,手动告诉他需要更新
            flag_modified(equipment, "attr")
            db.session.commit()

        except Exception as e:
            logger.exception(e)

```



## 参考
https://stackoverflow.com/questions/42559434/updates-to-json-field-dont-persist-to-db
https://docs.sqlalchemy.org/en/13/core/type_basics.html#sqlalchemy.types.JSON

