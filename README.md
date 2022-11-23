# mitosite-form

Универсальная форма с отправкой сообщений на емейл. Используются: библиотека PHPMailer, SMTP, JS, AJAX. Валидация HTML+JS, защиты от ботов, сообщения об успехе/ошибке.

---

## Требования к PHP-файлу

- PHP-файл с настройками (обработчик) должен быть один универсальный для всех вариантов отправки сообщений на почту. Он должен поддерживать любые варианты инпутов: текстовые, чекбоксы, радио, селекты, рейнджи и отправку файлов.

- Единственные места где нужно вносить изменения для нового проекта - это емейл отправителя, пароль отправителя, емейл получателя и заголовки-подписи для писем.

- Письма должны отправляться самым лучшим и надежным способом. Насколько знаю это SMTP с указанием емейла отправителя + пароль для приложений к нему. Желательно чтобы емейл отправителя и получателя был олдин и тот же.

- После отправки сообщения нужно чтобы пользователя не перекидывало со страницы с формой, чтобы эта страница не перезагружалась, чтобы JS-скрипт открывал сообщения об успешной или неуспешной отправке. А если скрипта нету, то в идеале добиться максимально похожего поведения с помощью только PHP-скрипта.

- Для отправки файлов вероятно стоит добавить грубую валидацию, чтобы пользователи не могли переслать откровенно огромные или опасные файлы. Для этого ограничить размер вложения 200 Мб и форматы файлов (например указать разрешенные).

---

## Текущие проблемы конфига mail.php

Дорабатывать вероятно нужно два участка кода, это "Формирование самого письма":
```
//$title = "Тема письма";
$file = $_FILES['file'];
$c = true;

// Формирование самого письма:
$title = "Сообщение из Mitorun Studio";
$body = "";
foreach ( $_POST as $key => $value ) {
	if ( $value != "" && $key != "project_name" && $key != "admin_email" && $key != "form_subject" ) {
		$body .= "
		" . ( ($c = !$c) ? '<tr>':'<tr style="background-color: #f8f8f8;">' ) . "
			<td style='padding: 10px; border: #e9e9e9 1px solid;'><b>$key</b></td>
			<td style='padding: 10px; border: #e9e9e9 1px solid;'>$value</td>
		</tr>
		";
	}
}
$body = "<table style='width: 100%;'>$body</table>";
```
Тут есть лишний код: прописано условие, что $key сравнивается с project_name, admin_email и form_subject. Автор говорит что это код со старого скрипта и это условие можно удалить.

Еще вопросы вызывает частое использование $body и $title - нужны ли они?


...и "Отправка сообщения":
```
// Отправка сообщения:
	$mail->isHTML(true);
	$mail->Subject = $title;
	$mail->Body = $body;
	$mail->send();
} catch (Exception $e) {
	$status = "Сообщение не было отправлено. Причина ошибки: {$mail->ErrorInfo}";
}
```

---

<br><br><br><br><br>

## Требования к управляющему JS-скрипту

- Модульность: чтобы каждую отдельную функцию можно было просто удалить или наоборот добавить (например код инпута-ловушки, код маски или валидации) и при этом скрипт продолжал работать с остальными функциями.

- Форма должна работать без перезагрузки страницы, оставаться на этой странице, введенная информация не должна удаляться или мигать пока форма успешно не отправится.

- После отправки формы скрипт должен показать сообщение об успешной или неудачной отправке. Форма должна очиститься.

- Работа нескольких форм на странице, включая в модалке (от одних и тех же скриптов JS/PHP и от разных).

- Код современный, лаконичный, поддерживаемый всеми современными браузерами с возрастом до 2 лет (IE не надо).

- Желательно встроить в скрипт простые и полезные функции безопасности, например инпут-приманка, может есть еще что-то?

### Подробнее что сделать в JS-скрипте

#### Скрипт для инпута-приманки

Что отправка работала и с приманкой и без приманки, а когда приманка заполнена, то письмо чтобы не отправлялось. Чтобы можно было просто добавлять или удалять в форме этот инпут, а в скрипте чтобы можно было легко удалить этот участок кода. На бэкенде ваш скрипт обработки может проверить, заполнено ли поле-приманка. Если это так, то отправка, вероятно, была от бота, и можно ее проигнорировать.

```
<label class="honeypot" for="honeypot" style="display: none;" aria-hidden="true" hidden>Оставьте это поле пустым</label>
<input class="honeypot" id="honeypot" style="display: none;" type="text" autocomplete="off" tabindex="-1" aria-hidden="true" hidden>
```

#### Сообщение об успехе/неудаче отправки

- После успешной отправки сообщения скрипт должен показать пользователю что сообщение отправлено и очистить форму. Не знаю какое тут лучшее решение. Сейчас в HTML добавляю:
```
<p class="form-success">Сообщение отправлено.</p>
<p class="form-failure">Сообщение не отправлено!</p>
```
И скрипт должен добавить к этим элементам класс ".active" чтобы они появились. Как вариант можно скрыть кнопку отправки и на ее месте показать сообщение об успехе, тогда макет не будет сдвигаться (но надо продумать хорошо ли это, как будет доступность?).

#### Скрипт для отправки файлов внедрить в общий скрипт отправки

Чтобы он работал когда есть поле отправки и не мешал когда его нет. Чтобы имел базовую валидацию отправляемых файлов: максимальный размер, форматы файлов, количество файлов? Или валидацию лучше делать в PHP-скрипте?


