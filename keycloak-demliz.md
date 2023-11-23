Работы по keycloak в ООО "Демлиз"
=================================

Версии keycloak 12.0.4, 18.0.2  

1.UserStorage Provider для Единой Базы пользователей (ЕБП)  
------------------------------------------------------------  

Основные функции:  
 - Поиск пользователей в ЕБП по различным атрибутам  
 - Проверка пароля в ЕБП  
 - Изменение пароля в ЕБП  
 - Добавление нового пользователя
 - Удаление пользователя
 - Изменение атрибутов пользователя
 - Получение справочников из ЕБП: Справочник государств, Проживание, Отрасль знаний, Категория читателя
 
Дополнительно:   
 - Scope rsl_udb_profile для вывода атрибутов профиля ЕБП в userinfo для OIDC  
 - Mapper атрибутов ЕБП uid, reader_number и personScopedAffiliation для SAML клиента  
 uid -> saml_udb_uid,  
 reader_number -> saml_udb_reader_number,  
 !readerNumber.isBlank()?"reader@rsl.ru":"non-reader@rsl.ru" -> personScopedAffiliation
 
2.Identity Provider для ЕСИА  
-----------------------------  

Основные функции:  
 - Авторизация в ЕСИА  
 - Получение информации из профиля пользователя ЕСИА:  
 OID  
 ФИО  
 Дата рождения  
 Пол  
 Гражданство  
 Контакты (/ctts): EML - электронная почта, MBT - мобильный телефон, PHN - домашний телефон  
 Адреса (/addrs): PLV - адрес места проживания, PRG - адрес места регистрации  
 Документы (/docs): RF_PASSPORT - паспорт гражданина РФ
   
Провайдер может работать с разными ЕСИА clientId в зависимости от домена с которого заходили: .rsl.ru - AIBS02771, .rusneb.ru - NELB01771. Соответственно, использует разные сертификаты и приватные ключи. Это настраивается в конфиге.  

Скоупы: openid email fullname birthdate gender id_doc contacts
 
Провайдер определяет три типа учётных записей ЕСИА:  
 - SIMPLE - упрощенная учетная запись - rIdDoc отсутствует, trusted=false  
 - STANDART - стандартная учетная запись - rIdDoc присутствует, trusted=false  
 - VERIFIED - подтвержденная учетная запись - rIdDoc присутствует, trusted=true  

Формы:  
First Broker Login Flow  
 - RSL Consent - Форма согласия на обработку персональных данных - login/rsl-consent.ftl  
 - UDB Review Profile - Форма для изменения полей пользователя ЕБП - login/login-update-udb-profile.ftl  
 - Create User If UDB Unique - Проверка уникальности ЕБП пользователя. Проверяет существование ЕБП аккаунта с таким же email, esia_id или mobile. Отправляет письмо с паролем и письмо об успешной регистрации на сайте РГБ на email нового пользователя.  

ЕСИА Post Login Flow  
 - Check Esia Type - Проверяет возможное изменение типа ЕСИА аккаунта (УЗ). Если тип УЗ изменился на "Подтверждённая" (verified), выводит форму login/postlogin-update-udb-profile.ftl с уведомлением об изменении типа УЗ и обновлении данных профиля в ЕБП из ЕСИА.  

3.Вход по СМС паролю
---------------------

Browser Flow  
 - SMS Password Authenticator - Вход по номеру мобильного телефона и СМС паролю. Находит пользователя по номеру телефона и отсылает ему СМС с кодом, используя сервис SMSAero.  

Форма ввода номера телефона и СМС пароля с google reCAPTCHA - login/sms-login.ftl

4.Письмо об успешной регистрации на сайте РГБ
----------------------------------------------

Registration Flow  
 - RslRegisteredEmailAuthenticator - Отсылает письмо об успешной регистрации на сайте РГБ на email нового пользователя.   

5.Селектор темы
---------------

FrontendUriThemeSelectorProvider  
Выбирает тему в зависимости от её типа и frontend URI, с которого заходили.  

EMailClientThemeSelectorProvider  
Возвращает для клиента OIDC ту же тему EMAIL, что и LOGIN из его настроек, если в настройках клиента выбрана тема LOGIN.

6.EmailSenderProvider
---------------------

NewEmailSenderProvider - EmailSenderProvider с возможностью брать email из метода getNewEmail() класса NewEmailUserModel    
NewEmailUserModel - Оболочка для org.keycloak.models.UserModel с дополнительным полем newEmail  
ChangeEmailActionToken - ActionToken с дополнительными полями newEmail и reduri  
ChangeEmailActionTokenHandler - обработчик для ChangeEmailActionToken  

7.Специальное REST API
----------------------

RslRestApiResourceProvider  
Основные функции:  
 - Информация о пользователе  
 - Обновление информации о пользователе  
 - Отсылка ChangeEmailAction на указанный email  
 - Изменение пароля пользователя  
 - Список связей пользователя с IDP  
 - Список IDP с информацией о связях с пользователем  
 - Создание связи с IDP  
 - Удаление связи с IDP  
 