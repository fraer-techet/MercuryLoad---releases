using System;
using System.Collections.ObjectModel;
using System.Diagnostics;
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
        public IntPtr WindowHandle { get; set; } // Уникальный номер окна в Windows
        public double IndicatorOpacity { get; set; } = 0.0; // 1.0 - светится, 0.0 - невидима
    }

    public partial class MainWindow : Window
    {
        // --- МАГИЯ WINDOWS API ДЛЯ РАБОТЫ С ОКНАМИ ---
        [DllImport("user32.dll")] private static extern int GetWindowLong(IntPtr hwnd, int index);
        [DllImport("user32.dll")] private static extern int SetWindowLong(IntPtr hwnd, int index, int newStyle);
        [DllImport("user32.dll")] private static extern bool EnumWindows(EnumWindowsProc enumProc, IntPtr lParam);
        [DllImport("user32.dll")] private static extern bool IsWindowVisible(IntPtr hWnd);
        [DllImport("user32.dll", CharSet = CharSet.Auto)] private static extern int GetWindowTextLength(IntPtr hWnd);
        [DllImport("user32.dll")] private static extern uint GetWindowThreadProcessId(IntPtr hWnd, out uint lpdwProcessId);
        [DllImport("user32.dll")] private static extern bool SetForegroundWindow(IntPtr hWnd);
        [DllImport("user32.dll")] private static extern bool ShowWindow(IntPtr hWnd, int nCmdShow);
        
        private delegate bool EnumWindowsProc(IntPtr hWnd, IntPtr lParam);

        private const int GWL_EXSTYLE = -20;
        private const int WS_EX_TOOLWINDOW = 0x00000080;
        private const int WS_EX_NOACTIVATE = 0x08000000;
        private const int SW_RESTORE = 9; // Команда развернуть окно

        // Список, который автоматически обновляет интерфейс
        private ObservableCollection<DockApp> openApps = new ObservableCollection<DockApp>();
        private DispatcherTimer scannerTimer = new DispatcherTimer();

        public MainWindow()
        {
            InitializeComponent();
            AppList.ItemsSource = openApps;

            // Настраиваем радар (он будет сканировать окна раз в секунду)
            scannerTimer.Interval = TimeSpan.FromSeconds(1);
            scannerTimer.Tick += ScannerTimer_Tick;
            scannerTimer.Start();
        }

        // --- ТОТ САМЫЙ СКАНЕР ОКОН ---
        private void ScannerTimer_Tick(object? sender, EventArgs e)
        {
            openApps.Clear(); // Временно очищаем док перед сканированием (позже сделаем плавно)

            // Просим Windows перечислить все-все окна
            EnumWindows(delegate (IntPtr hWnd, IntPtr lParam)
            {
                // Если окно невидимое или у него нет названия - пропускаем (это мусор)
                if (!IsWindowVisible(hWnd) || GetWindowTextLength(hWnd) == 0)
                    return true;

                // Узнаем, какому процессу принадлежит окно
                GetWindowThreadProcessId(hWnd, out uint processId);
                
                try
                {
                    Process proc = Process.GetProcessById((int)processId);
                    string path = proc.MainModule?.FileName ?? "";

                    // Отсеиваем саму нашу программу и системный мусор
                    if (!string.IsNullOrEmpty(path) && !path.Contains("MacDock") && !path.Contains("ApplicationFrameHost"))
                    {
                        openApps.Add(new DockApp
                        {
                            AppName = proc.ProcessName,
                            AppPath = path,
                            WindowHandle = hWnd,
                            IconImage = GetIconFromFile(path),
                            IndicatorOpacity = 1.0 // Включаем белую точку!
                        });
                    }
                }
                catch { } // Игнорируем защищенные системные процессы (антивирусы и т.д.)

                return true;
            }, IntPtr.Zero);
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

        // --- ЛОГИКА КЛИКА ПО ОТКРЫТОМУ ОКНУ ---
        private void AppIcon_Click(object sender, System.Windows.Input.MouseButtonEventArgs e)
        {
            var border = sender as FrameworkElement;
            var app = border?.DataContext as DockApp;

            if (app != null)
            {
                // Если окно уже существует - разворачиваем его и выводим на передний план!
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
