# Домашнее задание к занятию «1.2. Популярные языки, системы сборки, управления зависимостями»

В качестве результата отправьте ответы на вопросы в личном кабинете студента на сайте [netology.ru](https://netology.ru).

## Предисловие

ДЗ будет представлять собой лабораторные работы, в которых вы по инструкциям выполните определённые шаги.

## Задание SonarQube

На лекции мы с вами говорили, что исходный код приложения — это источник потенциальных уязвимостей.

Конечно же, исходный код приложения можно проверить и глазами, но при современных объёмах кода — это достаточно трудоемкая задача.

Поэтому существуют специальные инструменты, которые позволяют анализировать качество кода, в том числе пытаются найти в нём уязвимости.

С одним из подобных инструментов ([SonarQube](https://www.sonarqube.org/)) мы познакомимся в этом ДЗ, альтернативы рассмотрим на одной из следующих лекций.

**Важно**: вам не нужно учить Java и детально разбираться в коде. Ваши задачи:
1. Получить базовый опыт работы с инструментом.
1. Проанализировать предупреждения, баги и уязвимости.

### Описание проекта

Мы подготовили для вас учебный проект, написанный на языке Java и использующий систему сборки Maven.

Проект представляет собой веб-сервер, работающий на порту 8080 и отвечающий HTTP-запросам.

Страница http://localhost:8080/users.html закрыта логином и паролем admin/secret.

Чтобы собрать образ и запустить его (это необязательно для выполнения ДЗ), вам нужно:
1. Скачать [app.tgz](assets/app.tgz).
1. Скачать [Dockerfile](assets/Dockerfile).
1. Скачать [docker-compose.yml](assets/docker-compose.yml).
1. В каталоге со скачанными файлами выполнить: `docker-compose up --build ibdev`.

### Порядок выполнения

**Важно**: если вы планируете работать в Play With Docker, то используйте `tmux` или подключение по ssh для открытия нескольких терминалов. Или, если вы хорошо владеете консолью, переводите foreground процессы в background и обратно по мере необходимости.

0\. Скопируйте на целевую машину файл [docker-compose.yml](assets/docker-compose.yml) и откройте терминал в том же каталоге.

1\. Запустите сервис SonarQube:

```shell
docker-compose up sonarqube
```

Команда позволяет запустить только этот сервис и те, от которых он зависит (а не все перечисленные в `docker-compose.yml`).

2\. Дождитесь появления в логах записи `SonarQube is up`, после чего зайдите на http://localhost:9000 (или `docker-machine ip` и порт 9000, или на 9000 порту в Play With Docker).

3\. Для входа используйте следующие учётные данные:
* логин — `admin`
* пароль — `admin`

4\. На главной странице нажмите кнопку `Create new project`:

![](pic/sonar01.png)

5\. Введите ключ проекта (это нечто вроде кода проекта) и нажмите кнопку `Setup`:

![](pic/sonar02.png)

6\. Введите имя токена (ключа доступа) и нажмите на кнопку `Generate`:

![](pic/sonar03.png)

7\. После генерации токена нажмите на кнопку `Continue`:

![](pic/sonar04.png)

8\. Выберите опцию `Maven` и скопируйте сгенерированный код:

![](pic/sonar05.png)

Необходимо скопировать вот этот код:
```
mvn sonar:sonar \
  -Dsonar.projectKey=ru.netology:ibdev \
  -Dsonar.host.url=http://ip172-18-0-13-c02u36plo55000cqjg9g-9000.direct.labs.play-with-docker.com \
  -Dsonar.login=2a820a891192340e52fbfb011c706ec798b03c76
```

И заменить в нём строку: `-Dsonar.host.url=http://ip172-18-0-13-c02u36plo55000cqjg9g-9000.direct.labs.play-with-docker.com \` на `-Dsonar.host.url=http://sonarqube:9000 \`.

Т. е. должно получиться:
```
mvn sonar:sonar \
  -Dsonar.projectKey=ru.netology:ibdev \
  -Dsonar.host.url=http://sonarqube:9000 \
  -Dsonar.login=2a820a891192340e52fbfb011c706ec798b03c76
```

Это сделано потому, что контейнер с SonarQube и Maven будут находиться в одной сети, созданной для них Docker Compose, в которой сетевой доступ к сервисам возможно осуществлять по их имени.

**Q**: что делает эта команда?

**A**: большинство систем сборки (мы используем Maven) расширяемы за счёт использования плагинов (специальных дополнений). В этом случае SonarQube предоставляет для Maven Plugin, который называется `sonar`, и его задача — интегрируясь в Maven, проанализировать проект и отправить результаты анализа в SonarQube.

**Важно**: это команда для sh. Т. е. вы можете её использовать в sh, bash, Cygwin Terminal. Если вы работаете в CMD или PowerShell, то уберите все `\` и переносы строки и пишите всё в одну строку.

9\. Откройте новый терминал в том же каталоге, где у вас расположен файл `docker-compose.yml`.

10\. Выполните следующие команды:

```shell
# скачиваем архив с приложением
wget https://raw.githubusercontent.com/netology-code/ibdev-homeworks/master/02_dev/assets/app.tgz
# распаковываем архив
tar -xvf app.tgz
```

11\. Запустите [контейнер со сборочной системой Maven](https://hub.docker.com/_/maven): 

```shell
docker-compose run maven mvn -f /app sonar:sonar \
      -Dsonar.projectKey=ru.netology:ibdev \
      -Dsonar.host.url=http://sonarqube:9000 \
      -Dsonar.login=2a820a891192340e52fbfb011c706ec798b03c76
```

Обратите внимание: начиная с `sonar:sonar` - это то, что вы скопировали на шаге 8, с тем изменением, что после `mvn` добавлена опция `-f /app`.

12\. Дождитесь появления сообщения об окончании процесса:

![](pic/sonar-finished.png)

13\. Перейдите в веб-интерфейс SonarQube:

![](pic/sonar-results.png)

14\. Перейдите в раздел Bugs (ошибки в программном коде):

![](pic/sonar-results-bugs.png)

В этом разделе для каждой записи вы можете:
1. Изменить тип: Bug, Vulnerability, Code Smell.
1. Установить приоритет: Blocker — блокирует всю работу над продуктом, Critical — критичный, Major — важный, Minor — неважный, Info — информационное сообщение.
1. Статус: Open — открыт, Resolve as fixed — «закрыть» как исправлено, Resolve as false positive — «закрыть» как ложное срабатывание, Resolve as won't fix — «закрыть» как непланируемое к исправлению.
1. Увидеть примерную оценку по времени на исправление.
1. Установить комментарий.

15\. Перейдите в раздел Vulnerabilities (уязвимости в программном коде):

![](pic/sonar-results-vulnerabilities.png)

В этом разделе для каждой записи вы можете:
1. Изменить тип: Bug, Vulnerability, Code Smell.
1. Установить приоритет: Blocker — блокирует всю работу над продуктом, Critical — критичный, Major — важный, Minor — неважный, Info — информационное сообщение.
1. Статус: Open — открыт, Resolve as fixed — «закрыть» как исправлено, Resolve as false positive — «закрыть» как ложное срабатывание, Resolve as won't fix — «закрыть» как непланируемое к исправлению.
1. Увидеть примерную оценку по времени на исправление.
1. Установить комментарий.

16\. Перейдите в раздел Security Hotspots (код, требующий ручного анализа на предмет наличия уязвимости):

![](pic/sonar-results-security-hotspots.png)

В этом разделе для каждой записи вы можете:
1. Выставить статус.
1. Назначить ответственного.

17\. Перейдите в раздел Code Smells (запахи кода — признаки плохого кода):

![](pic/sonar-results-code-smells.png)

В этом разделе для каждой записи вы можете:
1. Изменить тип: Bug, Vulnerability, Code Smell.
1. Установить приоритет: Blocker — блокирует всю работу над продуктом, Critical — критичный, Major — важный, Minor — неважный, Info — информационное сообщение).
1. Статус: Open — открыт, Resolve as fixed — «закрыть» как исправлено, Resolve as false positive — «закрыть» как ложное срабатывание, Resolve as won't fix — «закрыть» как непланируемое к исправлению.
1. Увидеть примерную оценку по времени на исправление.
1. Установить комментарий.

### Результаты выполнения

![](001_02.jpg)
Отправьте в личном кабинете студента ответы на следующие вопросы:
1. Какие баги были выявлены: количество, описание, почему SonarQube их считает багами? См. ссылку `Why is this an issue?`.
Выявлен один баг.
```
 private static void initDb(Connection conn, boolean clean) {
    clean = true;
```
Introduce a new variable instead of reusing the parameter "clean".
Параметр clean перезаписывается внутри метода, что может привести к путанице. Это нарушение принципов чистого кода

1. Какие уязвимости были выявлены: количество, категории, описание, почему SonarQube их считает уязвимостями?
Выявлена одна уязвимость
```
 private static Connection getConnection() throws SQLException {
    return DriverManager.getConnection("jdbc:h2:~/h2.db", "sa", "");
```
Использование пустого пароля для доступа к базе данных считается серьезной уязвимостью

1. Какие Security Hotspots были выявлены: количество, категории, приоритет, описание, почему SonarQube их считает Security HotSpot'ами?
Выявлен один Security Hostpots:
```
      server.start();
      Runtime.getRuntime().addShutdownHook(new Thread(server::shutdown));
      Thread.currentThread().join();
    } catch (Exception e) {
      e.printStackTrace();
    }

```
Связана с использованием Runtime.getRuntime().addShutdownHook
Добавление shutdown hook может привести к утечке ресурсов, если хук не будет корректно удален или если он будет выполнять операции, которые могут заблокировать завершение работы приложения
Если злоумышленник получит возможность добавить свой собственный shutdown hook, он может выполнить вредоносный код при завершении работы приложения
Shutdown hooks выполняются в неопределенном порядке, что может привести к непредсказуемому поведению
Исключения выводятся в стандартный поток ошибок (e.printStackTrace()), что может быть недостаточно для корректной обработки ошибок.

Приоритет низкий

1. К каким CWE идёт отсылка для Security Hotspots из п. 2? См. вкладку `How can you fix it?` в нижней части страницы.

CWE-521 - Weak Password Requirements

1. Какие запахи кода были выявлены: количество, описание, почему SonarQube их считает запахами кода? См. ссылку `Why is this an issue?`.

Выявлено 5 Code Smell:
```
import java.io.IOException;
```
Импорт java.io.IOException не используется в коде. Это классический пример неиспользуемого импорта


```
 final var authorization = request.getAuthorization();
          System.out.println(authorization);
```
Используется System.out.println для вывода информации. Это нарушает рекомендации по написанию качественного кода

```
  private static void initDb(Connection conn, boolean clean) {
    clean = true;
    try (
        final var stmt = conn.createStatement();
    ) {
      if (clean) {
```
Переменной clean явно присваивается true а потом зачем то проверяется условие. Код в данном случае с if бессмыслен

```
catch (SQLException e) {
      throw new RuntimeException(e);
```

Используется универсальное исключение (RuntimeException) вместо специализированного исключения. Это нарушает рекомендации по написанию качественного и поддерживаемого кода.

```
catch (SQLException e) {
      throw new RuntimeException(e);
```
Аналогично
