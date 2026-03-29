using System;
using System.Collections.Generic;
using System.Collections.ObjectModel;
using System.ComponentModel;
using System.Diagnostics;
using System.IO;
using System.Linq;
using System.Runtime.InteropServices;
using System.Text;
using System.Windows;
using System.Windows.Controls;
using System.Windows.Interop;
using System.Windows.Media;
using System.Windows.Media.Imaging;
using System.Windows.Threading;

namespace MacDock
{
    // Теперь наша модель умная! Она умеет сообщать интерфейсу, если текст или точка поменялись
    public class DockApp : INotifyPropertyChanged
    {
        public string AppName { get; set; } = string.Empty;
        public string AppPath { get; set; } = string.Empty;
        public ImageSource? IconImage { get; set; }
        public IntPtr WindowHandle { get; set; } 

        private double indicatorOpacity = 0.0;
        public double IndicatorOpacity 
        { 
            get => indicatorOpacity; 
            set { indicatorOpacity = value; OnPropertyChanged(nameof(IndicatorOpacity)); }
        }

        private bool isPinned;
        public bool IsPinned
        {
            get => isPinned;
            set { isPinned = value; OnPropertyChanged(nameof(IsPinned)); OnPropertyChanged(nameof(PinText)); }
        }

        // Текст кнопки меняется в зависимости от того, закреплено приложение или нет
        public string PinText => IsPinned ? "Открепить" : "Закрепить в доке";

        public event PropertyChangedEventHandler? PropertyChanged;
        protected void OnPropertyChanged(string name) => PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(name));
    }

    public partial class MainWindow : Window
    {
        // --- ФИЛЬТРЫ И УПРАВЛЕНИЕ ОКНАМИ ---
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
        [DllImport("user32.dll")] private static extern IntPtr GetForegroundWindow(); // Проверка активного окна
        [DllImport("dwmapi.dll")] private static extern int DwmGetWindowAttribute(IntPtr hwnd, int dwAttribute, out int pvAttribute, int cbAttribute);

        private delegate bool EnumWindowsProc(IntPtr hWnd, IntPtr lParam);

        private const int GWL_EXSTYLE = -20;
        private const int WS_EX_TOOLWINDOW = 0x00000080; 
        private const int WS_EX_APPWINDOW = 0x00040000;  
        private const int WS_EX_NOACTIVATE = 0x08000000;
        private const uint GW_OWNER = 4;
        private const int DWMWA_CLOAKED = 14;
        private const int SW_MINIMIZE = 6; // Команда свернуть
        private const int SW_RESTORE = 9;  // Команда развернуть

        private ObservableCollection<DockApp> dockItems = new ObservableCollection<DockApp>();
        private DispatcherTimer scannerTimer = new DispatcherTimer();
        private string pinnedFilesPath = "pinned.txt"; // Файл для сохранения закрепленных прог

        public MainWindow()
        {
            InitializeComponent();
            AppList.ItemsSource = dockItems;

            LoadPinnedApps(); // Загружаем сохраненные иконки при старте

            scannerTimer.Interval = TimeSpan.FromMilliseconds(500);
            scannerTimer.Tick += ScannerTimer_Tick;
            scannerTimer.Start();
        }

        // --- УМНЫЙ СКАНЕР ОКОН ---
        private void ScannerTimer_Tick(object? sender, EventArgs e)
        {
            List<string> currentActivePaths = new List<string>();
            Dictionary<string, IntPtr> pathTopWindow = new Dictionary<string, IntPtr>();
            Dictionary<string, string> pathTopTitle = new Dictionary<string, string>();

            EnumWindows(delegate (IntPtr hWnd, IntPtr lParam)
            {
                if (IsTaskbarWindow(hWnd))
                {
                    GetWindowThreadProcessId(hWnd, out uint processId);
                    try
                    {
                        Process proc = Process.GetProcessById((int)processId);
                        string path = proc.MainModule?.FileName ?? "";

                        if (!string.IsNullOrEmpty(path) && !path.Contains("MacDock"))
                        {
                            // Группируем окна: берем только первое найденное окно программы
                            if (!currentActivePaths.Contains(path))
                            {
                                currentActivePaths.Add(path);
                                pathTopWindow[path] = hWnd;

                                StringBuilder title = new StringBuilder(GetWindowTextLength(hWnd) + 1);
                                GetWindowText(hWnd, title, title.Capacity);
                                pathTopTitle[path] = title.ToString();
                            }
                        }
                    } catch { } 
                }
                return true;
            }, IntPtr.Zero);

            // 1. Убираем незакрепленные программы, которые закрыли
            for (int i = dockItems.Count - 1; i >= 0; i--)
            {
                if (!currentActivePaths.Contains(dockItems[i].AppPath) && !dockItems[i].IsPinned)
                    dockItems.RemoveAt(i);
            }

            // 2. Обновляем статус всех иконок (светится точка или нет)
            foreach (var app in dockItems)
            {
                if (currentActivePaths.Contains(app.AppPath))
                {
                    app.WindowHandle = pathTopWindow[app.AppPath];
                    app.AppName = pathTopTitle[app.AppPath];
                    app.IndicatorOpacity = 1.0; // Включаем точку
                }
                else
                {
                    app.WindowHandle = IntPtr.Zero; // Окно закрыто
                    app.IndicatorOpacity = 0.0;     // Выключаем точку
                }
            }

            // 3. Добавляем новые запущенные программы
            foreach (var path in currentActivePaths)
            {
                if (!dockItems.Any(a => a.AppPath == path))
                {
                    dockItems.Add(new DockApp {
                        AppPath = path, AppName = pathTopTitle[path], WindowHandle = pathTopWindow[path],
                        IndicatorOpacity = 1.0, IsPinned = false, IconImage = GetIconFromFile(path)
                    });
                }
            }
        }

        // --- ЛОГИКА КЛИКА: СВЕРНУТЬ ИЛИ РАЗВЕРНУТЬ? ---
        private async void AppIcon_Click(object sender, System.Windows.Input.MouseButtonEventArgs e)
        {
            var border = sender as FrameworkElement;
            var app = border?.DataContext as DockApp;

            if (app != null && border != null)
            {
                // Анимация прыжка
                var transformGroup = border.RenderTransform as TransformGroup;
                var translate = transformGroup?.Children[1] as TranslateTransform;
                if (translate != null)
                {
                    System.Windows.Media.Animation.DoubleAnimation bounceAnim = new System.Windows.Media.Animation.DoubleAnimation {
                        To = -10, Duration = TimeSpan.FromMilliseconds(150), AutoReverse = true, RepeatBehavior = new System.Windows.Media.Animation.RepeatBehavior(1)
                    };
                    translate.BeginAnimation(TranslateTransform.YProperty, bounceAnim);
                }

                await Task.Delay(100);

                if (app.WindowHandle == IntPtr.Zero)
                {
                    // Окно закрыто, но закреплено -> Запускаем заново!
                    try { Process.Start(new ProcessStartInfo(app.AppPath) { UseShellExecute = true }); } catch { }
                }
                else
                {
                    // Проверяем, активно ли окно прямо сейчас
                    IntPtr activeWindow = GetForegroundWindow();
                    
                    if (activeWindow == app.WindowHandle)
                    {
                        // Окно активно -> Сворачиваем!
                        ShowWindow(app.WindowHandle, SW_MINIMIZE);
                    }
                    else
                    {
                        // Окно свернуто или под другим окном -> Разворачиваем!
                        ShowWindow(app.WindowHandle, SW_RESTORE);
                        SetForegroundWindow(app.WindowHandle);
                    }
                }
            }
        }

        // --- ЛОГИКА ЗАКРЕПЛЕНИЯ ПРИЛОЖЕНИЙ ---
        private void PinApp_Click(object sender, RoutedEventArgs e)
        {
            var menuItem = sender as MenuItem;
            var app = menuItem?.DataContext as DockApp;

            if (app != null)
            {
                app.IsPinned = !app.IsPinned; // Переключаем статус
                SavePinnedApps();             // Сохраняем в файл
            }
        }

        private void SavePinnedApps()
        {
            // Собираем пути всех закрепленных программ и пишем в файл
            var pinnedPaths = dockItems.Where(a => a.IsPinned).Select(a => a.AppPath);
            File.WriteAllLines(pinnedFilesPath, pinnedPaths);
        }

        private void LoadPinnedApps()
        {
            if (File.Exists(pinnedFilesPath))
            {
                foreach (var path in File.ReadAllLines(pinnedFilesPath))
                {
                    if (File.Exists(path) && !dockItems.Any(a => a.AppPath == path))
                    {
                        dockItems.Add(new DockApp {
                            AppPath = path, AppName = Path.GetFileNameWithoutExtension(path),
                            IsPinned = true, IndicatorOpacity = 0.0, // Закреплено, но закрыто (точки нет)
                            WindowHandle = IntPtr.Zero, IconImage = GetIconFromFile(path)
                        });
                    }
                }
            }
        }

        // --- ВСПОМОГАТЕЛЬНЫЕ ФУНКЦИИ ---
        private bool IsTaskbarWindow(IntPtr hWnd)
        {
            if (!IsWindowVisible(hWnd)) return false;
            int titleLength = GetWindowTextLength(hWnd);
            if (titleLength == 0) return false;

            DwmGetWindowAttribute(hWnd, DWMWA_CLOAKED, out int cloaked, sizeof(int));
            if (cloaked != 0) return false;

            int exStyle = GetWindowLong(hWnd, GWL_EXSTYLE);
            IntPtr owner = GetWindow(hWnd, GW_OWNER);

            if ((exStyle & WS_EX_TOOLWINDOW) != 0 && (exStyle & WS_EX_APPWINDOW) == 0) return false;
            if (owner != IntPtr.Zero && (exStyle & WS_EX_APPWINDOW) == 0) return false;

            StringBuilder title = new StringBuilder(titleLength + 1);
            GetWindowText(hWnd, title, title.Capacity);
            string windowTitle = title.ToString();
            
            if (windowTitle == "Program Manager" || windowTitle == "Settings") return false;

            return true; 
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
            } catch { } return null;
        }

        private void ExitMenu_Click(object sender, RoutedEventArgs e) => System.Windows.Application.Current.Shutdown();

        private void Window_SourceInitialized(object? sender, EventArgs e)
        {
            IntPtr hwnd = new WindowInteropHelper(this).Handle;
            SetWindowLong(hwnd, GWL_EXSTYLE, GetWindowLong(hwnd, GWL_EXSTYLE) | WS_EX_TOOLWINDOW | WS_EX_NOACTIVATE);
        }

        private void Window_Loaded(object sender, RoutedEventArgs e)
        {
            this.Left = (SystemParameters.PrimaryScreenWidth - this.Width) / 2;
            this.Top = SystemParameters.PrimaryScreenHeight - this.Height;
        }
    }
}
