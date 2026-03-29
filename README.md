using System;
using System.Runtime.InteropServices;
using System.Windows;
using System.Windows.Interop;

namespace MacDock
{
    public partial class MainWindow : Window
    {
        // === ИМПОРТ ФУНКЦИЙ WINDOWS API ===
        // Эти функции позволяют нам менять глубокие настройки окна
        [DllImport("user32.dll")]
        private static extern int GetWindowLong(IntPtr hwnd, int index);

        [DllImport("user32.dll")]
        private static extern int SetWindowLong(IntPtr hwnd, int index, int newStyle);

        // Константы Windows API
        private const int GWL_EXSTYLE = -20;
        private const int WS_EX_TOOLWINDOW = 0x00000080; // Скрывает из Alt+Tab
        private const int WS_EX_NOACTIVATE = 0x08000000; // Окно не забирает фокус при клике

        public MainWindow()
        {
            InitializeComponent();
        }

        // 1. Этот метод вызывается, когда окно получает свой "системный номер" (Handle),
        // но еще не отрисовалось на экране. Идеальное место для применения хаков WinAPI.
        private void Window_SourceInitialized(object sender, EventArgs e)
        {
            // Получаем уникальный номер (Handle) нашего окна
            IntPtr hwnd = new WindowInteropHelper(this).Handle;

            // Читаем текущие стили окна
            int extendedStyle = GetWindowLong(hwnd, GWL_EXSTYLE);

            // Добавляем стили "Не забирать фокус" и "Скрыть из Alt+Tab"
            SetWindowLong(hwnd, GWL_EXSTYLE, extendedStyle | WS_EX_TOOLWINDOW | WS_EX_NOACTIVATE);
        }

        // 2. Этот метод вызывается, когда окно уже готово показаться.
        // Здесь мы рассчитываем его позицию на экране.
        private void Window_Loaded(object sender, RoutedEventArgs e)
        {
            PositionDock();
        }

        // Метод для умного позиционирования дока
        private void PositionDock()
        {
            // SystemParameters.WorkArea возвращает размер экрана МИНУС панель задач Windows.
            // Благодаря этому док не будет залезать на панель задач, если она у тебя внизу.
            Rect workArea = SystemParameters.WorkArea;

            // Ставим окно ровно по центру по горизонтали
            this.Left = workArea.Left + (workArea.Width - this.Width) / 2;
            
            // Прижимаем окно к самому низу доступной рабочей области
            this.Top = workArea.Bottom - this.Height;
        }
    }
}
