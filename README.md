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
        [DllImport("user32.dll")]
        private static extern int GetWindowLong(IntPtr hwnd, int index);
        [DllImport("user32.dll")]
        private static extern int SetWindowLong(IntPtr hwnd, int index, int newStyle);

        private const int GWL_EXSTYLE = -20;
        private const int WS_EX_TOOLWINDOW = 0x00000080;
        private const int WS_EX_NOACTIVATE = 0x08000000;

        public MainWindow()
        {
            InitializeComponent();
            
            // ПОДУШКА БЕЗОПАСНОСТИ: Если при загрузке будет ошибка, она выведет ее на экран!
            try
            {
                LoadApps();
            }
            catch (Exception ex)
            {
                MessageBox.Show("Ошибка при загрузке иконок: " + ex.Message, "ОШИБКА", MessageBoxButton.OK, MessageBoxImage.Error);
            }
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
                        IconImage = GetIconFromFile(path)
                    });
                }
            }

            AppList.ItemsSource = myApps;
        }

        // --- САМЫЙ НАДЕЖНЫЙ МЕТОД ИЗВЛЕЧЕНИЯ ИКОНОК ---
        private ImageSource? GetIconFromFile(string filePath)
        {
            try
            {
                // Используем легальный метод, который мы только что разблокировали в .csproj
                using (System.Drawing.Icon? sysIcon = System.Drawing.Icon.ExtractAssociatedIcon(filePath))
                {
                    if (sysIcon != null)
                    {
                        return Imaging.CreateBitmapSourceFromHIcon(
                            sysIcon.Handle,
                            Int32Rect.Empty,
                            BitmapSizeOptions.FromEmptyOptions());
                    }
                }
            }
            catch { } // Если файл без иконки - просто вернет пустоту, БЕЗ ВЫЛЕТА
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
