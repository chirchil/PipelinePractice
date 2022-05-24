# Проверим тесты
Установлю pytest 
 ```shell
   pip install pytest
```
и запущу тесты

![img](screens/Screenshot_105.png)

Все тесты пройдены.

Теперь допущу ошибку в классе Calculator и проверю тесты 

![img](screens/Screenshot_106.png)

... сломались.

Для функционального тестирования установлю selenium
```shell
pip install selenium
```
Запущу веб-приложение
```shell
python app.py
``` 
и тесты
```shell
py.test tests/functional_tests
```

![img](screens/Screenshot_107.png)

полет нормальный. Теперь сломаю преобразование в инт переменных для суммирования

![img](screens/Screenshot_108.png)

Упала ошибка. Возвращаю все к прежнему состоянию и пушу в гит.

# Переходим к Azure

Захожу в вами созданную организацию

![img](screens/Screenshot_113.png)

Создаю новый пайплайн и указываю git как источник кода

![img](screens/Screenshot_110.png)

В yaml файл вставляю следующий код (триггерю на мастер и девелоп)
```yaml
# Python package
# Create and test a Python package on multiple Python versions.
# Add steps that analyze code, save the dist with the build record, publish to a PyPI-compatible index, and more:
# https://docs.microsoft.com/azure/devops/pipelines/languages/python

trigger: # в данном блоке определяется, при каком событии запускать пайплайн
- master # запускаем, как только пришел новый коммит в ветку master
- develop # запускаем, как только пришел новый коммит в ветку develop

pool: # здесь определяем образ докера, в котором запускается приложение и версию интерпритатора
    vmImage: ubuntu-latest # выбираем ubuntu
strategy: # здесь выбираем стек программирования
    matrix: # matrix позволяет запускать параллельные конвейеры, если требуются разные версии
#    Python36: # пока отключаем запуск на 3.6
#      python.version: '3.6'
    Python37: # запускаем на 3.7
        python.version: '3.7'

steps: # здесь указываются шаги конвейера
- task: UsePythonVersion@0 # используем версию питона
    inputs:
    versionSpec: '$(python.version)'
    displayName: 'Use Python $(python.version)'

- script: | # запускаем апдейт питона, устанавливаем зависимости (в нашем случае flask)
    python -m pip install --upgrade pip
    pip install -r requirements.txt
    displayName: 'Install dependencies' # здесь отображается название текущей задачи

- script: | # запускаем юнит тесты (без функциональных)
    pip install pytest 
    pytest tests/unit_tests && pip install pycmd && py.cleanup tests/
    displayName: 'pytest'

- task: ArchiveFiles@2 # архивируем наш проект чтобы опубликовать артефакт. Артефакт это по сути то, что отдает клиенту (например архив с программой)
    displayName: 'Archive application'
    inputs:
    rootFolderOrFile: '$(System.DefaultWorkingDirectory)/'
    includeRootFolder: false
    archiveFile: '$(Build.ArtifactStagingDirectory)/$(Build.BuildId).zip'

- task: PublishBuildArtifacts@1 # публикуем артефакт как результат нашего пайплайна
    displayName: 'Publish Artifact: drop'

```

Запускаю пайплайн, смотрю за результатами и проверяю, что все выполнилось корректно

![img](screens/Screenshot_115.png)

Артефакт:

![img](screens/Screenshot_116.png)

И добавляю себе в гит бейдж

![img](screens/Screenshot_117.png)

# Continious deployment
Создал ресурсную группу

![img](screens/Screenshot_120.png)

Далее создаю Release и в него добавляю ранее упомянутый артефакт

![img](screens/Screenshot_122.png)

Следующим шагом добавляю deploy 

![img](screens/Screenshot_128.png)

И нажимаю create release 

![img](screens/Screenshot_127.png)

Смотрим результаты

![img](screens/Screenshot_129.png)
![img](screens/Screenshot_130.png)

Настрою функциональное тестирование

Добалвю новый stage 'test'

![img](screens/Screenshot_131.png)

Со следующим сценарием и параметрами

![img](screens/Screenshot_132.png)
![img](screens/Screenshot_133.png)
![img](screens/Screenshot_135.png)

Создаю release

![img](screens/Screenshot_142.png)

Все тесты выполнились

![img](screens/Screenshot_138.png)

![img](screens/Screenshot_139.png)

На гитхабе сделаю тестовый коммит и вижу, что тесты прошли

![img](screens/Screenshot_113.png)

## Так же я попал в число тех, кому не повезло с колбочкой на Azure, поэтому на этом все!




