---
title: База данных SQLite Provider - ограничения — EF Core
author: rowanmiller
ms.date: 04/09/2017
ms.assetid: 94ab4800-c460-4caa-a5e8-acdfee6e6ce2
uid: core/providers/sqlite/limitations
ms.openlocfilehash: eaa7d5b1496172e4f3821433a1cd098ee7e8b737
ms.sourcegitcommit: 9bd64a1a71b7f7aeb044aeecc7c4785b57db1ec9
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 06/26/2019
ms.locfileid: "67394806"
---
# <a name="sqlite-ef-core-database-provider-limitations"></a>Ограничения функций поставщика базы данных SQLite EF Core

Поставщик SQLite имеет ряд ограничений миграции. Большинство из этих ограничений представляют собой результат применения ограничения в ядро СУБД SQLite, а не только к EF.

## <a name="modeling-limitations"></a>Ограничения моделирования

Общей библиотекой реляционных (совместно используемые поставщиками реляционной базы данных Entity Framework) определяет API-интерфейсы для моделирования основные понятия, которые являются общими для большинства реляционных СУБД. Несколько из этих понятий, не поддерживаются поставщиком SQLite.

* Схемы
* Последовательности
* Вычисляемые столбцы

## <a name="query-limitations"></a>Ограничения запросов

SQLite изначально не поддерживают следующие типы данных. EF Core может считывать и записывать значения этих типов, и запрос для проверки на равенство (`where e.Property == value`) также поддерживается. Другие операции, однако, как и сравнение и упорядочение потребует оценка на клиенте.

* DateTimeOffset
* Десятичное число
* TimeSpan
* UInt64

Вместо `DateTimeOffset`, мы рекомендуем использовать значения даты и времени. При обработке нескольких часовых поясов, рекомендуется преобразовать значения в формат UTC перед сохранением и затем преобразуется в соответствующий часовой пояс.

`Decimal` Тип обеспечивает высокий уровень точности. Если вам не понадобиться такой уровень точности, тем не менее, рекомендуется вместо этого использовать double. Можно использовать [преобразователь значений](../../modeling/value-conversions.md) продолжать использовать decimal в классах.

``` csharp
modelBuilder.Entity<MyEntity>()
    .Property(e => e.DecimalProperty)
    .HasConversion<double>();
```

## <a name="migrations-limitations"></a>Ограничения миграции

Ядро СУБД SQLite поддерживает ряд операций схемы, которые поддерживаются в большинстве других реляционных баз данных. При попытке применить одну из неподдерживаемых операций с базой данных SQLite то `NotSupportedException` будет создано.

| Операция            | Поддерживается? | Требуется версия |
|:---------------------|:-----------|:-----------------|
| AddColumn            | ✔          | 1.0              |
| AddForeignKey        | ✗          |                  |
| AddPrimaryKey        | ✗          |                  |
| AddUniqueConstraint  | ✗          |                  |
| AlterColumn          | ✗          |                  |
| CreateIndex          | ✔          | 1.0              |
| CreateTable          | ✔          | 1.0              |
| DropColumn           | ✗          |                  |
| DropForeignKey       | ✗          |                  |
| DropIndex            | ✔          | 1.0              |
| DropPrimaryKey       | ✗          |                  |
| DropTable            | ✔          | 1.0              |
| DropUniqueConstraint | ✗          |                  |
| RenameColumn         | ✔          | 2.2.2            |
| RenameIndex          | ✔          | 2.1              |
| RenameTable          | ✔          | 1.0              |
| EnsureSchema         | ✔ (нет-op)  | 2.0              |
| DropSchema           | ✔ (нет-op)  | 2.0              |
| Insert               | ✔          | 2.0              |
| Обновление               | ✔          | 2.0              |
| Оператор delete               | ✔          | 2.0              |

## <a name="migrations-limitations-workaround"></a>Инструкции по решению ограничения миграции

Вы можете устранить некоторые из этих ограничений, написав код в миграции для выполнения таблицы вручную перестроить. Перестройка таблицы включает в себя переименование существующей таблицы, создание новой таблицы, копирование данных в новую таблицу и удаление старой таблицы. Необходимо будет использовать `Sql(string)` метод для выполнения некоторых из этих действий.

См. в разделе [внесения других типов из таблицы изменений схемы](http://sqlite.org/lang_altertable.html#otheralter) в SQLite документации для получения дополнительных сведений.

В будущем EF может поддерживать некоторые из этих операций, используя подход перестроение таблицы, на самом деле. Вы можете [отслеживать эту функцию на проект на портале GitHub](https://github.com/aspnet/EntityFrameworkCore/issues/329).
