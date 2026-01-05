# Chloros+ Вход

## Chloros и Chloros (браузер) Вход

Пользовательское <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> боковое меню позволяет вам войти в свою учетную запись Chloros+ и разблокировать дополнительные функции.

После входа в систему будут отображены данные вашей учетной записи:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>## CLI Вход

Войдите с помощью своих учетных данных Chloros+, чтобы включить обработку CLI.

**Синтаксис:**

```bash
chloros-cli login <email> <password>
```

{% hint style=&quot;info&quot; %}
**Пользователи SDK**: Python SDK также предоставляет программный метод `logout()` для очистки кэшированных учетных данных. Подробности см. в [документации Python SDK](api-python-sdk.md#logout).
{% endhint %}

**Пример:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style=&quot;warning&quot; %}
**Специальные символы**: используйте одинарные кавычки вокруг паролей, содержащих такие символы, как `$`, `!` или пробелы.
{% endhint %}

**Вывод:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>### Истечение срока действия плана

Срок истечения плана в графическом интерфейсе пользователя показывает, когда ваша лицензия станет недействительной. Для ежемесячных подписок срок истечения наступает в конце месяца. Для годовых подписок — через год после начала подписки. Для проверки лицензии требуется ежемесячное подключение к Интернету с 30-дневным льготным периодом.

### Ограничение по количеству устройств

Каждый план Chloros+ предлагает разное количество зарегистрированных устройств. Каждое устройство, на которое вы входите с учетной записью Chloros+, будет учитываться в количестве зарегистрированных устройств. Вы можете переименовать и удалить устройство на странице своей учетной записи MAPIR Cloud.

<table><thead><tr><th width="168.5999755859375" align="right">План Chloros</th><th align="center">COPPER</th><th align="center">БРОНЗ</th><th align="center">СЕРЕБРО</th><th align="center">ЗОЛОТО</th></tr></thead><tbody><tr><td align="right">Поддерживаемые устройства</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>
