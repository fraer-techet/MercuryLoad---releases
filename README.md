using System;
using System.Collections.Generic;
using System.Collections.ObjectModel;
using System.Diagnostics;
using System.Linq;
using System.Runtime.InteropServices;
using System.Text;
using System.Windows;
using System.Windows.Interop;
using System.Windows.Media;
using System.Windows.Media.Imaging;
using System.Windows.Threading;

namespace MacDock
{
    public class DockApp
    {
        public string AppName { get; set; } = string.Empty;
        public string AppPath { get; set; } = string.Empty;
        public ImageSource? IconImage { get; set; }
        public IntPtr WindowHandle { get; set; } 
        public double IndicatorOpacity { get; set; } = 1.0; 
    }

    public partial class MainWindow : Window
    {
        // --- ФИЛЬТРЫ И ИНСТРУМЕНТЫ WINDOWS API ---
        [DllImport("user32.dll")] private static extern int GetWindowLong(IntPtr hwnd, int index);
        [DllImport("user32.dll")] private static extern int SetWindowLong(IntPtr hwnd, int index, int newStyle);
        [DllImport("user32.dll")] private static extern bool EnumWindows(EnumWindowsProc enumProc, IntPtr lParam);
        [DllImport("user32.dll")] private static extern bool IsWindowVisible(IntPtr hWnd);
        [DllImport("user32.dll", CharSet = CharSet.Auto)] private static extern int GetWindowTextLength(IntPtr hWnd);
        [DllImport("user32.dll", CharSet = CharSet.Auto)] private static extern int GetWindowText(IntPtr hWnd, StringBuilder lpString, int nMaxCount);
        [DllImport("user32.dll")] private static extern uint GetWindowThreadProcessId(IntPtr hWnd, out uint lpdwProcessId);
        [DllImport("user32.dll")] private static extern bool SetForegroundWindow(IntPtr hWnd);
        [DllImport("user32.dll")] private static extern bool ShowWindow(IntPtr hWnd, int nCmdShow);
        [DllImport("user32.dll")] private static extern IntPtr GetWindow(IntPtr hWnd, uint uCmd);
        
        // Для Windows 10/11 (отсеиваем окна-"призраки" от UWP приложений)
        [DllImport("dwmapi.dll")] private static extern int DwmGetWindowAttribute(IntPtr hwnd, int dwAttribute, out int pvAttribute, int cbAttribute);

        private delegate bool EnumWindowsProc(IntPtr hWnd, IntPtr lParam);

        private const int GWL_EXSTYLE = -20;
        private const int WS_EX_TOOLWINDOW = 0x00000080; // Скрытые системные тултипы
        private const int WS_EX_APPWINDOW = 0x00040000;  // Принудительно на панели задач
        private const int WS_EX_NOACTIVATE = 0x08000000;
        private const uint GW_OWNER = 4;
        private const int DWMWA_CLOAKED = 14;
        private const int SW_RESTORE = 9;

        // Наш УМНЫЙ список (сам обновляет интерфейс без миганий)
        private ObservableCollection<DockApp> openApps = new ObservableCollection<DockApp>();
        private DispatcherTimer scannerTimer = new DispatcherTimer();

        public MainWindow()
        {
            InitializeComponent();
            AppList.ItemsSource = openApps;

            scannerTimer.Interval = TimeSpan.FromMilliseconds(500); // Сканируем каждые полсекунды
            scannerTimer.Tick += ScannerTimer_Tick;
            scannerTimer.Start();
        }

        // --- УМНЫЙ СКАНЕР ПАНЕЛИ ЗАДАЧ ---
        private void ScannerTimer_Tick(object? sender, EventArgs e)
        {
            List<IntPtr> currentWindows = new List<IntPtr>();

            // 1. Собираем все правильные окна
            EnumWindows(delegate (IntPtr hWnd, IntPtr lParam)
            {
                if (IsTaskbarWindow(hWnd))
                {
                    currentWindows.Add(hWnd);
                }
                return true;
            }, IntPtr.Zero);

            // 2. УДАЛЯЕМ из дока те программы, которые ты закрыл
            for (int i = openApps.Count - 1; i >= 0; i--)
            {
                if (!currentWindows.Contains(openApps[i].WindowHandle))
                {
                    openApps.RemoveAt(i);
                }
            }

            // 3. ДОБАВЛЯЕМ только новые программы (без перерисовки старых!)
            foreach (var hWnd in currentWindows)
            {
                // Если такого окна еще нет в доке
                if (!openApps.Any(app => app.WindowHandle == hWnd))
                {
                    GetWindowThreadProcessId(hWnd, out uint processId);
                    try
                    {
                        Process proc = Process.GetProcessById((int)processId);
                        string path = proc.MainModule?.FileName ?? "";

                        // Защита от попадания нашего собственного дока
                        if (!string.IsNullOrEmpty(path) && !path.Contains("MacDock"))
                        {
                            StringBuilder title = new StringBuilder(GetWindowTextLength(hWnd) + 1);
                            GetWindowText(hWnd, title, title.Capacity);

                            openApps.Add(new DockApp
                            {
                                AppName = title.ToString(), // Теперь пишет реальное название окна (напр. "YouTube - Google Chrome")
                                AppPath = path,
                                WindowHandle = hWnd,
                                IconImage = GetIconFromFile(path),
                                IndicatorOpacity = 1.0
                            });
                        }
                    }
                    catch { } // Игнорируем защищенные системные процессы
                }
            }
        }

        // --- ТОТ САМЫЙ АЛГОРИТМ ФИЛЬТРАЦИИ МУСОРА ---
        private bool IsTaskbarWindow(IntPtr hWnd)
        {
            // 1. Окно должно быть видимым
            if (!IsWindowVisible(hWnd)) return false;

            // 2. У окна должно быть название
            int titleLength = GetWindowTextLength(hWnd);
            if (titleLength == 0) return false;

            // 3. Отсекаем окна-призраки (особенность Windows 11)
            DwmGetWindowAttribute(hWnd, DWMWA_CLOAKED, out int cloaked, sizeof(int));
            if (cloaked != 0) return false;

            int exStyle = GetWindowLong(hWnd, GWL_EXSTYLE);
            IntPtr owner = GetWindow(hWnd, GW_OWNER);

            // 4. Логика Microsoft: 
            // Если это ToolWindow (системная всплывашка) - скрываем.
            if ((exStyle & WS_EX_TOOLWINDOW) != 0 && (exStyle & WS_EX_APPWINDOW) == 0) return false;
            
            // Если у окна есть "Хозяин" (например, окно сохранения файла внутри браузера) - скрываем, показываем только главное.
            if (owner != IntPtr.Zero && (exStyle & WS_EX_APPWINDOW) == 0) return false;

            // 5. Исключаем стандартные системные скрытые элементы
            StringBuilder title = new StringBuilder(titleLength + 1);
            GetWindowText(hWnd, title, title.Capacity);
            string windowTitle = title.ToString();
            
            if (windowTitle == "Program Manager" || windowTitle == "Settings") return false;

            return true; // Если прошло все проверки - это НАСТОЯЩЕЕ окно!
        }

        private ImageSource? GetIconFromFile(string filePath)
        {
            try
            {
                using (System.Drawing.Icon? sysIcon = System.Drawing.Icon.ExtractAssociatedIcon(filePath))
                {
                    if (sysIcon != null)
                        return Imaging.CreateBitmapSourceFromHIcon(sysIcon.Handle, Int32Rect.Empty, BitmapSizeOptions.FromEmptyOptions());
                }
            }
            catch { }
            return null;
        }

        private void AppIcon_Click(object sender, System.Windows.Input.MouseButtonEventArgs e)
        {
            var border = sender as FrameworkElement;
            var app = border?.DataContext as DockApp;

            if (app != null)
            {
                // Прыжок иконки перед разворачиванием
                var transformGroup = border.RenderTransform as TransformGroup;
                var translate = transformGroup?.Children[1] as TranslateTransform;
                if (translate != null)
                {
                    System.Windows.Media.Animation.DoubleAnimation bounceAnim = new System.Windows.Media.Animation.DoubleAnimation {
                        To = -10, Duration = TimeSpan.FromMilliseconds(150), AutoReverse = true, RepeatBehavior = new System.Windows.Media.Animation.RepeatBehavior(1)
                    };
                    translate.BeginAnimation(TranslateTransform.YProperty, bounceAnim);
                }

                // Разворачиваем окно
                ShowWindow(app.WindowHandle, SW_RESTORE);
                SetForegroundWindow(app.WindowHandle);
            }
        }

        private void ExitMenu_Click(object sender, RoutedEventArgs e)
        {
            System.Windows.Application.Current.Shutdown();
        }

        private void Window_SourceInitialized(object? sender, EventArgs e)
        {
            IntPtr hwnd = new WindowInteropHelper(this).Handle;
            int extendedStyle = GetWindowLong(hwnd, GWL_EXSTYLE);
            SetWindowLong(hwnd, GWL_EXSTYLE, extendedStyle | WS_EX_TOOLWINDOW | WS_EX_NOACTIVATE);
        }

        private void Window_Loaded(object sender, RoutedEventArgs e)
        {
            this.Left = (SystemParameters.PrimaryScreenWidth - this.Width) / 2;
            this.Top = SystemParameters.PrimaryScreenHeight - this.Height;
        }
    }
}
