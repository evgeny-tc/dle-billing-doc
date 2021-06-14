# DLE Billing

[![Version](https://img.shields.io/badge/Version-0.7.4-blue.svg?style=flat-square "Version")](https://github.com/Japing/dle-billing/releases)
![DLE](https://img.shields.io/badge/DLE-13.0--14.2-green.svg?style=flat-square "DLE")
![Charset](https://img.shields.io/badge/Charset-utf--8-red.svg?style=flat-square "Charset")
![jQuery](https://img.shields.io/badge/jQuery-1.11+-yellow.svg?style=flat-square "jQuery")

Автоматизируйте приём платежей на сайте с помощью модуля DLE Billing. Предоставте пользователям возможность оплачивать различные товары и услуги вашего сайта.

 **Автор модуля:** mr_Evgen (evgeny.tc@gmail.com)  
 **Адаптировал:** Japing

### **Плагины**
- [Платный переход в группу](https://github.com/Japing/Paygroup-DLE-Billing)
- [Оплата скрытого текста](https://github.com/Japing/Payhide-DLE-Billing)

### **Системные требования**
- Версия DataLife Engine 13.0 - 14.2
- jQuery v1.11 и выше
- ЧПУ (mod_rewrite)
- Кодировка по умолчанию: UTF-8

### **Установка**
1. [Скачать архив](https://github.com/Japing/dle-billing/releases "Скачать архив") с модулем
2. Заходить в **Админ панель** -> **Утилиты** -> **Управление плагинами** и загрузить скачанный архив
3. Открыть страницу Ваш_Сайт.ру/admin.php?mod=billing
4. Выполнить инструкции установщика
5. Настроить модуль

### **Обновление**
1. Обновите модуль через систему плагинов
2. Войдите в админ.панель модуля для автоматического обновления бд и файла настроек.

------------
### Список изменений

#### v0.7.4 (25.08.2019)
- Модуль полностью адаптирован к системе плагинов.
- В личный кабинет пользователя добавлен раздел **«Квитанции»**, в котором отображается список всех квитанций.
- В настройках модуля добавлен новый пункт, указывающий максимальное количество неоплаченных квитанций которые может создать пользователь.
- В личный кабинет пользователя добавлена возможность удалять неоплаченные квитанции.
- Исправлена ошибка при отображении статистики в админ панель модуля.
- Изменён принцип создание новых квитанций: теперь создается квитанция и её можна оплачивать любой платежной системой.

#### v0.7.3 (06.06.2019)
- Модуль адаптирован под DLE 13.0 и выше .
- Изменены иконки в админ панель модуля.

[========]

## Документация
### Подключение новых биллингов
Файлы платежных системы (ПС) находятся в каталоге **/engine/modules/billing/payments/**:

| Файл / каталог  | Описание  |
| ------------ | ------------ |
|../payments/name/  | Каталог, содержащий файлы каждой отдельной ПС  |
|info.ini| Файл конфигурации с информацией о ПС  |
|adm.settings.php| Главный файл ПС  |
|/icon/icon.png|Каталог, содержащий иконку ПС для отображения в админпанели  |

#### info.ini
    [info]
    version=1.0
    title=Название
    desc=Краткое описание платежной системы

#### adm.settings.php
Данный файл содержит класс **Payment** со следующими обязательными методами:

```php
Class Payment
```

##### Settings( ... )
возвращает массив настроек для редактирования в админ.панели
```php
/*
	 $config - массив с текущими настройками ПС
*/
function Settings( $config ) 
{
  $Form = array();
      
  $Form[] = array("Параметр #1:", "Описание параметра #1", "<input name='save_con[param1]' type='text' value='" . $config['param1'] ."'>" );
  $Form[] = array("Параметр #2:", "Описание параметра #2", "<input name='save_con[param2]' type='text' value='" . $config['param2'] ."'>" );
      
  return $Form;
}
```

##### Form( ... )
возвращает форму с данными платежа для отправки на сайт ПС
```php
/*
	 $id - id квитанции, уникален для каждой операции
	 $config- настройки ПС из админ.панели
	 $invoice - массив с квитанцией на оплату (таблица бд dle_billing_invoice)
	 $currency - валюта платежа (после склонения)
	 $desc - описание платежа
*/
function form( $id, $config, $invoice, $currency, $desc ) 
{
   return '
   "<form method="post" id="paysys_form" action="https://merchant.webmoney.ru/lmi/payment.asp">
   "<input name="lmi_payment_desc" value="'.$desc.'" type="hidden">
   "<input name="lmi_payment_no" value="'.$id.'" type="hidden">
   "<input name="lmi_payment_amount" value="'.$invoice['invoice_pay'].'" type="hidden">
   "<input name="lmi_sim_mode" value="0" type="hidden">
   "<input name="lmi_payee_purse" value="'.$config['wm'].'" type="hidden">
   "<input type="submit" class="bs_button" value="Оплатить">
   "</form>';
}
```

##### Check_id( ... )
метод должен выбрать ID квитанции из данных, полученных от сервера ПС
```php
/*
	 $data - массив с данными, полученными от сервера ПС
*/
function check_id( $data ) 
{
  return $DATA['m_orderid'];
}
```

##### Check_ok( ... )
метод выводит сообщения на запрос ПС о статусе сайта в случаи успешной оплаты
```php
/*
	 $data - массив с данными, полученными от сервера ПС
*/
function check_ok( $data )
{
  header( $_SERVER['SERVER_PROTOCOL'] . ' HTTP 200 OK', true, 200 );
  
  echo $DATA['m_orderid'] . '|success';
    
  return true;
}
```

##### Check_out( ... )
метод проверяет принятые от ПС данные. В случае успешной проверки - возвращает 200, в ином случае - сообщение об ошибке
```php
/*
	 $data - массив с данными, полученными от сервера ПС
	 $config - настройки ПС из админпанели
	 $invoice - массив с квитанцией на оплату (таблица бд dle_billing_invoice)
*/
function check_out( $data, $config, $invoice ) 
{
  if (isset($data['m_operation_id']) && isset($data['m_sign']))
  {
    $arHash = array($data['m_operation_id'],
        $data['m_operation_ps'],
        $data['m_operation_date'],
        $data['m_operation_pay_date'],
        $data['m_shop'],
        $data['m_orderid'],
        $data['m_amount'],
        $data['m_curr'],
        $data['m_desc'],
        $data['m_status'],
        $config['secret_key']);
          
    $sign_hash = strtoupper(hash('sha256', implode(':', $arHash)));
      
    if ($data['m_sign'] == $sign_hash && $data['m_status'] == 'success')
    {
      return 200;
    }
    return $data['m_orderid'].'|error';
  }
  
  return 'Data|error';
}
```

##### Check_payer_requisites( ... )
метод выбирает реквизиты покупателя из массива данных от ПС
```php
/*
	 $data - массив с данными, полученными от сервера ПС
*/
function check_payer_requisites( $data )
{
	return 'WMID' . $data["LMI_PAYER_WM"];
}
```

### Режим тестирования
При возникновении проблем с приёмом платежей:

1. Включите Режим тестирования (Баланс пользователя → Настройки → Безопасность)
3. Попытайтесь пополнить баланс через интерфейс пользователя
5. Просмотреть результат обработки платежа Вы можете на странице Лог входящих запросов (сайт.ру/admin.php?mod=billing&m=log)

#### Расшифровка записей

| Запись  |  Описание |
| :------------ | :------------ |
|  Start: ### |  Получен запрос на result url для платёжной системы ### |
| Get data: serialize({array})  | Массив полученных данных от ПС  |
|  Test billing: OK	|  Платежная система доступна |
| Load adm.settings.php: OK / NO  |   Главный файл платёжной системы загружен / не найден|
| Error: get ID invoice  |  Ошибка, ID квитанции не получен (метод Check_id( … )) |
| Test ID invoice ({###}): OK  |  ID квитанции получен |
| Test billing hash: ###  |  Проверка контрольной подписи. В случае успеха - 200, иначе - описание ошибки |
| Send money: OK  |  Платеж прошел, пользователю зачислены средста на баланс |
| Error: ###  |  Возникла ошибка при проверке платежа. <br> Возможны варианты <br>Ключ доступа платежной системы устарел <br>Платежная система не найдена <br>Квитанция не найдена, либо уже оплачена |
| End  | Скрипт завершил работу без ошибок  |

Обязательно очистите лог кнопкой Сбросить.

### Файлы шаблонов

| Файл / каталог  | Описание  |
| :------------ | :------------ |
| billing.tpl| 	Шаблон для подключения модуля| 
| /billing/	| | 
| cabinet.tpl| 	Личный кабинет пользователя| 
| history.tpl| 	История движения средств| 
| msg.tpl| 	Системное сообщение| 
| /pay/| 	Каталог, содержащий шаблоны этапов пополнения баланса| 
| start.tpl| 	Страница пополнения баланса| 
| waiting.tpl| 	Просмотр счёта. Ожидания оплаты| 
| ok.tpl| 	Просмотр счёта. Оплата выполнена| 
| success.tpl| 	Баланс пополнен| 
| fail.tpl| 	Ошибка пополнения баланса| 
| /mail/| 	Каталог, содержащий шаблоны персональных уведомлений| 
| balance.tpl| 	Ваш баланс был изменён| 
| new.tpl| 	Новая счёт на оплату| 
| payok.tpl| 	Счёт оплачен| 
| /js/	| | 
| scripts.js| 	Главный js файл модуля| 
| /css/	| | 
| styles.css| 	Главный css файл модуля| 
| /icons/| 	Каталог, содержащий иконки платёжных систем| 
| /plugins/| 	Каталог, содержащий шаблоны плагинов| 
| refund.tpl| 	Возврат средств| 
| transfer.tpl| 	Перевод средств| 

### Персональные уведомления
Включите нужные уведомления в админпанели модуля → Настройки → Персональные уведомления

### API
Вы можете подключить платежи в ваш модуль, использая API:

Подключите файл api в ваш модуль:
```php
include ('engine/modules/billing/OutAPI.php');
```

Список функций:
##### Пополнить баланс
```php
/*
	Пополнить баланс пользователя mr_Evgen на 100.00 у.е. с описанием Подарок
*/
$BillingAPI->PlusMoney( "mr_Evgen", "100.00", "Подарок" );

/*
	Расширенная запись
	Пополнить баланс пользователя mr_Evgen на 100.00 у.е. с описанием Подарок
	в истории платежей указать тег операции: api и номер: 18
*/
$BillingAPI->PlusMoney( "mr_Evgen", "100.00", "Подарок", 'api, 18 );
```

##### Списать с баланса
```php
/*
	Cнять 50.00 у.е. с баланса пользователя mr_Evgen с описанием Оплата комментария
*/
$BillingAPI->MinusMoney( "mr_Evgen", "50.00", "Оплата комментария" );

/*
	Расширенная запись
	Cнять 50.00 у.е. с баланса пользователя mr_Evgen с описанием Оплата комментария
	в истории платежей указать тег операции: api и номер: 19
	допустить отрицательный баланс на счете (по умолчанию - нет)
*/
$BillingAPI->MinusMoney( "mr_Evgen", "50.00", "Оплата комментария", 'api', 19, false );
```

##### Отправить уведомление
```php
/*
	Массив с уведомлением
*/
$dataMail = array
(
   '{id}' => 1,
   '{summa}' => "200.00$",
   '{login}' => "mr_Evgen"
);
 
/*
	Отправить сообщение пользователю в лс и на email
	Использовать шаблон /mail/themeTPLname.tpl
*/
$BillingAPI->Alert( 'themeTPLname', $dataMail, 1, 'test@dle-billing.ru' );

/*
	Отправить сообщение пользователю в лс
*/
$BillingAPI->Alert( 'themeTPLname', $dataMail, 1);

/*
	Отправить сообщение пользователю на email
*/
$BillingAPI->Alert( 'themeTPLname', $dataMail, 0, 'test@dle-billing.ru');
```

##### Нумерация страниц
```php
/*
	Всего строк: 15
	Текущий номер страницы: 1
	Ссылка перехода по страницам: /billing.html/log/main/page/{p}
*/
$BillingAPI->Pagination( 15, 1, "/billing.html/log/main/page:{p}", "<a href='{page_num_link}'>{page_num}</a>", "<strong>{page_num}</strong>" );
```
##### Формат суммы
преобразование цены в формат, указанный в админ.панели
```php
// вернет "15.00" при формате данных - 0.00
$BillingAPI->Convert( 15 );
```

##### Наименование валюты
```php
// вернет "доллара"
$BillingAPI->Declension( 22.00, "доллар,доллара,долларов" );
```

