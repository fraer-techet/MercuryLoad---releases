using System;
using System.Collections.Generic;
using System.Diagnostics;
using System.Drawing; // Для работы с иконками
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
    // Модель нашего приложения (теперь с настоящей картинкой!)
    public class DockApp
    {
        public string AppName { get; set; }   // Название (для подсказки при наведении)
        public string AppPath { get; set; }   // Путь к .exe файлу
        public ImageSource IconImage { get; set; } // Настоящая картинка иконки
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
            LoadApps(); 
        }

        private void LoadApps()
        {
            var myApps = new List<DockApp>();

            // Список путей к стандартным программам (ты потом сможешь добавлять сюда ЛЮБЫЕ свои игры и проги)
            string[] defaultApps = {
                @"C:\Windows\explorer.exe", // Проводник
                @"C:\Windows\system32\notepad.exe", // Блокнот
                @"C:\Windows\system32\calc.exe", // Калькулятор
                @"C:\Windows\system32\cmd.exe" // Командная строка
            };

            foreach (string path in defaultApps)
            {
                if (File.Exists(path))
                {
                    myApps.Add(new DockApp
                    {
                        AppName = Path.GetFileNameWithoutExtension(path), // Берем название файла
                        AppPath = path,
                        IconImage = GetIconFromFile(path) // Магическая функция получения иконки!
                    });
                }
            }

            AppList.ItemsSource = myApps;
        }

        // --- ФУНКЦИЯ ИЗВЛЕЧЕНИЯ НАСТОЯЩИХ ИКОНОК ИЗ EXE ---
        private ImageSource GetIconFromFile(string filePath)
        {
            try
            {
                // Вытаскиваем иконку средствами Windows
                using (Icon sysIcon = Icon.ExtractAssociatedIcon(filePath))
                {
                    // Конвертируем в формат, понятный нашему красивому интерфейсу (WPF)
                    return Imaging.CreateBitmapSourceFromHIcon(
                        sysIcon.Handle,
                        Int32Rect.Empty,
                        BitmapSizeOptions.FromEmptyOptions());
                }
            }
            catch
            {
                return null; // Если иконки нет, вернет пустоту (но она всегда есть у exe)
            }
        }

        // --- ИДЕАЛЬНЫЙ, СОЧНЫЙ И КОРОТКИЙ ПРЫЖОК ПРИ КЛИКЕ ---
        private async void AppIcon_Click(object sender, System.Windows.Input.MouseButtonEventArgs e)
        {
            var border = sender as FrameworkElement;
            var app = border.DataContext as DockApp;

            if (app != null && border != null)
            {
                var transformGroup = border.RenderTransform as TransformGroup;
                var translate = transformGroup.Children[1] as TranslateTransform;

                // Аккуратный прыжок на 15 пикселей вверх
                DoubleAnimation bounceAnim = new DoubleAnimation {
                    To = -15, 
                    Duration = TimeSpan.FromMilliseconds(150), // Быстро вверх
                    AutoReverse = true, // И сразу плавно вниз
                    RepeatBehavior = new RepeatBehavior(1), 
                    EasingFunction = new QuadraticEase { EasingMode = EasingMode.EaseOut }
                };

                translate.BeginAnimation(TranslateTransform.YProperty, bounceAnim);

                // Даем 100 миллисекунд насладиться началом прыжка и запускаем программу!
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

        private void Window_SourceInitialized(object sender, EventArgs e)
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
