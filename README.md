using System;
using System.Collections.Generic;
using System.Diagnostics;
using System.IO;
using System.Runtime.InteropServices;
using System.Threading.Tasks;
using System.Windows;
using System.Windows.Interop;
using System.Windows.Media;
using System.Windows.Media.Animation;
using System.Windows.Media.Imaging;

namespace MacDock
{
    public class DockApp
    {
        public string AppName { get; set; } = string.Empty;   
        public string AppPath { get; set; } = string.Empty;   
        public ImageSource? IconImage { get; set; } 
    }

    public partial class MainWindow : Window
    {
        // --- БАЗОВЫЕ НАСТРОЙКИ ОКНА ---
        [DllImport("user32.dll")]
        private static extern int GetWindowLong(IntPtr hwnd, int index);
        [DllImport("user32.dll")]
        private static extern int SetWindowLong(IntPtr hwnd, int index, int newStyle);

        private const int GWL_EXSTYLE = -20;
        private const int WS_EX_TOOLWINDOW = 0x00000080;
        private const int WS_EX_NOACTIVATE = 0x08000000;

        // --- МАГИЯ ЯДРА WINDOWS ДЛЯ ИКОНОК ---
        [DllImport("shell32.dll", CharSet = CharSet.Auto)]
        private static extern IntPtr SHGetFileInfo(string pszPath, uint dwFileAttributes, ref SHFILEINFO psfi, uint cbSizeFileInfo, uint uFlags);

        [DllImport("user32.dll", SetLastError = true)]
        [return: MarshalAs(UnmanagedType.Bool)]
        private static extern bool DestroyIcon(IntPtr hIcon);

        [StructLayout(LayoutKind.Sequential, CharSet = CharSet.Auto)]
        private struct SHFILEINFO
        {
            public IntPtr hIcon;
            public int iIcon;
            public uint dwAttributes;
            [MarshalAs(UnmanagedType.ByValTStr, SizeConst = 260)]
            public string szDisplayName;
            [MarshalAs(UnmanagedType.ByValTStr, SizeConst = 80)]
            public string szTypeName;
        };

        private const uint SHGFI_ICON = 0x000000100;
        private const uint SHGFI_LARGEICON = 0x000000000;

        public MainWindow()
        {
            InitializeComponent();
            LoadApps(); 
        }

        private void LoadApps()
        {
            var myApps = new List<DockApp>();

            string[] defaultApps = {
                @"C:\Windows\explorer.exe",
                @"C:\Windows\system32\notepad.exe",
                @"C:\Windows\system32\calc.exe",
                @"C:\Windows\system32\cmd.exe"
            };

            foreach (string path in defaultApps)
            {
                if (File.Exists(path))
                {
                    myApps.Add(new DockApp
                    {
                        AppName = Path.GetFileNameWithoutExtension(path), 
                        AppPath = path,
                        IconImage = GetIconFromFile(path) // Теперь используем сверхнадежный метод
                    });
                }
            }

            AppList.ItemsSource = myApps;
        }

        // Новая функция, которая никогда не "уронит" программу
        private ImageSource? GetIconFromFile(string filePath)
        {
            try
            {
                SHFILEINFO shinfo = new SHFILEINFO();
                // Просим Windows отдать нам иконку этого файла
                IntPtr res = SHGetFileInfo(filePath, 0, ref shinfo, (uint)Marshal.SizeOf(shinfo), SHGFI_ICON | SHGFI_LARGEICON);
                
                if (res != IntPtr.Zero && shinfo.hIcon != IntPtr.Zero)
                {
                    // Конвертируем в формат WPF
                    ImageSource img = Imaging.CreateBitmapSourceFromHIcon(
                        shinfo.hIcon,
                        Int32Rect.Empty,
                        BitmapSizeOptions.FromEmptyOptions());
                    
                    // Очищаем память, чтобы док не "жрал" оперативку
                    DestroyIcon(shinfo.hIcon); 
                    return img;
                }
            }
            catch { }
            return null;
        }

        private async void AppIcon_Click(object sender, System.Windows.Input.MouseButtonEventArgs e)
        {
            var border = sender as FrameworkElement;
            var app = border?.DataContext as DockApp;

            if (app != null && border != null)
            {
                var transformGroup = border.RenderTransform as TransformGroup;
                var translate = transformGroup?.Children[1] as TranslateTransform;

                if (translate != null)
                {
                    DoubleAnimation bounceAnim = new DoubleAnimation {
                        To = -15, 
                        Duration = TimeSpan.FromMilliseconds(150), 
                        AutoReverse = true, 
                        RepeatBehavior = new RepeatBehavior(1), 
                        EasingFunction = new QuadraticEase { EasingMode = EasingMode.EaseOut }
                    };
                    translate.BeginAnimation(TranslateTransform.YProperty, bounceAnim);
                }

                await Task.Delay(100); 
                try {
                    Process.Start(new ProcessStartInfo(app.AppPath) { UseShellExecute = true });
                } catch { }
            }
        }

        private void ExitMenu_Click(object sender, RoutedEventArgs e)
        {
            Application.Current.Shutdown();
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
