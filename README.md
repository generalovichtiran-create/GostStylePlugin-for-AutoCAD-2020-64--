GostStylePlugin — автоматический ГОСТ-стиль текста для AutoCAD 2020
Готовый плагин, который избавляет от ручной настройки текстового стиля при каждом новом чертеже.

Если вы работаете с AutoCAD 2020 и устали каждый раз создавать стиль ГОСТ_Основной вручную, этот плагин сделает всё за вас. При создании нового документа он автоматически проверяет наличие стиля и, если его нет, создаёт с нужными параметрами.

🚀 Вариант 1: Скачать и использовать (ничего собирать не нужно)
Если вы не хотите связываться с Visual Studio и компиляцией, используйте готовый файл.

Шаг 1. Скачайте плагин
Перейдите на страницу Releases этого репозитория и скачайте последнюю версию файла GostStylePlugin.dll (или архив GostStylePlugin-v1.0.zip, внутри которого лежит DLL).

📌 Ссылка для вставки в ваш README: [Скачать GostStylePlugin.dll (Releases)](../../releases/latest)

Шаг 2. Загрузите плагин в AutoCAD
Откройте AutoCAD 2020.

В командной строке введите NETLOAD и нажмите Enter.

В открывшемся окне выберите скачанный файл GostStylePlugin.dll и нажмите «Открыть».

Шаг 3. Проверьте работу
Создайте новый документ (Ctrl+N). Плагин автоматически добавит текстовый стиль ГОСТ_Основной в ваш чертёж. Вы можете убедиться в этом, открыв диспетчер текстовых стилей (команда STYLE).

Также вы можете вызвать создание стиля вручную в любом чертеже командой GOSTStyle.

Шаг 4. (Необязательно) Настройте автозагрузку
Чтобы не загружать DLL вручную каждый раз при старте AutoCAD, настройте автоматическую загрузку через acaddoc.lsp:

Найдите папку поддержки AutoCAD (обычно C:\Program Files\Autodesk\AutoCAD 2020\Support).

Создайте или откройте файл acaddoc.lsp в блокноте.

Добавьте строку: (command "_NETLOAD" "C:/Путь/К/Вашему/GostStylePlugin.dll") .

Сохраните файл. Теперь плагин будет загружаться при каждом открытии или создании чертежа.

🛠️ Вариант 2: Собрать самостоятельно (подробная инструкция)
Этот раздел для тех, кто хочет понять, как всё устроено, или внести свои изменения в код.

Что понадобится
AutoCAD 2020 (64-бит).

Visual Studio Community (бесплатная версия). Скачать можно с официального сайта Microsoft .

.NET Framework 4.8.

Этап 1: Создание проекта в Visual Studio
Откройте Visual Studio и выберите «Создать новый проект».

В поиске введите Class Library (.NET Framework).

Выберите шаблон именно для C# (значок с фиолетовым C#). Не перепутайте с Visual Basic! Иначе компилятор выдаст сотни ошибок синтаксиса.

Назовите проект, например, GostStylePlugin.

Укажите расположение и выберите .NET Framework 4.8. Нажмите «Создать».

Этап 2: Добавление ссылок на библиотеки AutoCAD
Чтобы код «понимал» команды AutoCAD, нужно подключить его библиотеки.

В Обозревателе решений (справа) найдите папку «Ссылки» (References) или «Зависимости».

Нажмите на неё правой кнопкой мыши → «Добавить ссылку» (Add Reference).

Нажмите кнопку «Обзор» (Browse) и перейдите в папку установки AutoCAD: C:\Program Files\Autodesk\AutoCAD 2020\.

Если файлов не видно, проверьте подпапку Managed.

Найдите и выберите (с зажатой клавишей Ctrl) три файла:

AcDbMgd.dll

AcMgd.dll

AcCoreMgd.dll (этот файл критически важен, так как содержит Application.DocumentManager в современных версиях) .

Нажмите «Добавить», затем «ОК».

Этап 3: Критическая настройка «Copy Local»
Это нужно, чтобы Visual Studio не копировала библиотеки AutoCAD в папку с вашим плагином (иначе при загрузке в AutoCAD возникнет конфликт версий).

В «Обозревателе решений» разверните «Ссылки».

Выделите все три добавленные ссылки (AcDbMgd, AcMgd, AcCoreMgd).

Нажмите правой кнопкой → «Свойства».

Найдите пункт «Копировать локально» (Copy Local) и установите значение False для каждой.

Этап 4: Настройка архитектуры (x64)
AutoCAD 2020 — 64-битное приложение, поэтому плагин должен быть собран под ту же архитектуру.

Нажмите правой кнопкой на название вашего проекта (не решения) → «Свойства».

Перейдите на вкладку «Сборка» (Build).

Найдите поле «Целевая платформа» (Platform target) и смените значение с «Any CPU» на x64.

Этап 5: Вставка кода
Откройте файл Class1.cs (или Program.cs в зависимости от версии шаблона), удалите всё его содержимое и вставьте следующий код:

csharp
using Autodesk.AutoCAD.Runtime;
using Autodesk.AutoCAD.ApplicationServices;
using Autodesk.AutoCAD.DatabaseServices;
using Autodesk.AutoCAD.EditorInput;

namespace GostStylePlugin
{
    // Автоматический запуск при загрузке плагина
    public class GostStyleExtension : IExtensionApplication
    {
        public void Initialize()
        {
            // Подписываемся на событие создания документа
            Application.DocumentManager.DocumentCreated += OnDocumentCreated;
        }

        public void Terminate()
        {
            Application.DocumentManager.DocumentCreated -= OnDocumentCreated;
        }

        private void OnDocumentCreated(object sender, DocumentCollectionEventArgs e)
        {
            Document doc = e.Document;
            if (doc != null)
            {
                GostStyleHelper.EnsureGostStyle(doc);
            }
        }
    }

    // Логика создания стиля
    public static class GostStyleHelper
    {
        private const string StyleName = "ГОСТ_Основной";
        private const string FontFile = "ISOCPEUR.ttf"; // Встроенный аналог ГОСТ

        public static void EnsureGostStyle(Document doc)
        {
            Database db = doc.Database;
            Editor ed = doc.Editor;

            try
            {
                using (Transaction tr = db.TransactionManager.StartTransaction())
                {
                    TextStyleTable textStyleTable = (TextStyleTable)tr.GetObject(db.TextStyleTableId, OpenMode.ForRead);

                    if (!textStyleTable.Has(StyleName))
                    {
                        textStyleTable.UpgradeOpen();
                        TextStyleTableRecord newStyle = new TextStyleTableRecord();
                        newStyle.Name = StyleName;
                        newStyle.FileName = FontFile;
                        newStyle.TextSize = 0;
                        newStyle.XScale = 1.0;
                        // newStyle.ObliquingAngle = 75 * Math.PI / 180; // Наклон для ГОСТ тип А

                        textStyleTable.Add(newStyle);
                        tr.AddNewlyCreatedDBObject(newStyle, true);
                        db.Textstyle = newStyle.ObjectId;

                        ed.WriteMessage($"\nАвтоматически создан стиль '{StyleName}'.");
                    }
                    tr.Commit();
                }
            }
            catch (System.Exception ex)
            {
                ed.WriteMessage($"\nОшибка при создании стиля ГОСТ: {ex.Message}");
            }
        }
    }

    // Ручная команда (запасной вариант)
    public class GostStyleCommands
    {
        [CommandMethod("GOSTStyle")]
        public void ManualCreateGostStyle()
        {
            Document doc = Application.DocumentManager.MdiActiveDocument;
            GostStyleHelper.EnsureGostStyle(doc);
        }
    }
}
Этап 6: Сборка
В верхнем меню Visual Studio выберите «Сборка» → «Собрать решение» (Build → Build Solution). Если всё сделано правильно, в папке bin\x64\Debug\ (или bin\x64\Release\) появится файл GostStylePlugin.dll.

⚙️ Настройка под себя
В коде есть две ключевые константы:

StyleName — имя создаваемого стиля.

FontFile — имя файла шрифта. По умолчанию ISOCPEUR.ttf (встроен в AutoCAD). Если вы установили ГОСТ-шрифт (например, GOST type A.ttf), замените значение.

Для добавления наклона (как в ГОСТ тип А) раскомментируйте строку newStyle.ObliquingAngle = 75 * Math.PI / 180;.

📄 Лицензия
MIT. Используйте, модифицируйте и делитесь свободно.
