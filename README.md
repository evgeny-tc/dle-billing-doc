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

### **Установка и настройка модуля:**
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

##### Settings( ... )
возвращает массив настроек для редактирования в админ.панели
```php
function Settings( $config ) 
{
  $Form = array();
      
  $Form[] = array("Параметр #1:", "Описание параметра #1", "<input name='save_con[param1]' type='text' value='" . $config['param1'] ."'>" );
  $Form[] = array("Параметр #2:", "Описание параметра #2", "<input name='save_con[param2]' type='text' value='" . $config['param2'] ."'>" );
      
  return $Form;
}
```
- **$config** - массив с текущими настройками ПС.

##### Form( ... )
возвращает форму с данными платежа для отправки на сайт ПС
```php
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
- id - id квитанции, уникален для каждой операции
- config - настройки ПС из админпанели
- invoice - массив данных о квитанции:
- - invoice['invoice_paysys'] - название ПС лат.
- - invoice['invoice_user_name'] - логин пользователя
- - invoice['invoice_get'] - сумма к получению в валюте сайта
- - invoice['invoice_pay'] - сумма к оплате в валюте ПС
- - invoice['invoice_date_creat'] - дата и время создания квитанции (unixtime)
- - invoice['invoice_date_pay'] - дата и время оплаты (unixtime). Пока квитанция не оплачена = 0
- currency - валюта сайта
- desc - описание платежа

##### Check_id( ... )
метод должен выбрать ID квитанции из данных, полученных от сервера ПС
```php
function check_id( $data ) 
{
  return $DATA['m_orderid'];
}
```
- $data - массив с данными, полученными от сервера ПС.

##### Check_ok( ... )
метод выводит сообщения на запрос ПС о статусе сайта в случаи успешной оплаты
```php
function check_ok( $data )
{
  header( $_SERVER['SERVER_PROTOCOL'] . ' HTTP 200 OK', true, 200 );
  
  echo $DATA['m_orderid'] . '|success';
    
  return true;
}
```
- $data - массив с данными, полученными от сервера ПС.

##### Check_out( ... )
метод проверяет принятые от ПС данные. В случае успешной проверки - возвращает 200, в ином случае - сообщение об ошибке
```php
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
- data - массив с данными, полученными от сервера ПС.
- config - настройки ПС из админпанели
- invoice - массив данных о квитанции:
- - invoice['invoice_paysys'] - название ПС лат. (/BillingName/)
- - invoice['invoice_user_name'] - логин пользователя
- - invoice['invoice_get'] - сумма к получению в валюте сайта
- - invoice['invoice_pay'] - сумма к оплате в валюте ПС
- - invoice['invoice_date_creat'] - дата и время создания квитанции (unixtime)
- - invoice['invoice_date_pay'] - дата и время оплаты (unixtime). Пока квитанция не оплачена = 0

##### Check_payer_requisites( ... )
метод выбирает реквизиты покупателя из массива данных от ПС
```php
function check_payer_requisites( $data )
{
	return 'WMID' . $data["LMI_PAYER_WM"];
}
```
- $data - массив с данными, полученными от сервера ПС.
