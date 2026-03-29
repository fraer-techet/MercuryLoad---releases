using System;
using System.Collections.Generic;
using System.Diagnostics;
using System.Runtime.InteropServices;
using System.Windows;
using System.Windows.Interop;
using System.Windows.Media;
using System.Windows.Media.Animation; // Добавили для анимаций из кода
using System.Threading.Tasks;

namespace MacDock
{
    public class DockApp
    {
        public string Letter { get; set; }   
        public string AppPath { get; set; }  
        public Brush Color { get; set; }     
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
            var myApps = new List<DockApp>
            {
                new DockApp { Letter = "E", AppPath = "explorer.exe", Color = new SolidColorBrush((Color)ColorConverter.ConvertFromString("#E2B022")) }, 
                new DockApp { Letter = "W", AppPath = "msedge.exe", Color = new SolidColorBrush((Color)ColorConverter.ConvertFromString("#0078D7")) },   
                new DockApp { Letter = "N", AppPath = "notepad.exe", Color = new SolidColorBrush((Color)ColorConverter.ConvertFromString("#4CAF50")) },  
                new DockApp { Letter = "C", AppPath = "calc.exe", Color = new SolidColorBrush((Color)ColorConverter.ConvertFromString("#607D8B")) }      
            };

            AppList.ItemsSource = myApps;
        }

        // КЛИК И АНИМАЦИЯ ПРЫЖКА (BOUNCE)
        private async void AppIcon_Click(object sender, System.Windows.Input.MouseButtonEventArgs e)
        {
            var border = sender as System.Windows.FrameworkElement;
            var app = border.DataContext as DockApp;

            if (app != null && border != null)
            {
                // Находим трансформацию (TranslateTransform), которая отвечает за движение вверх-вниз
                var transformGroup = border.RenderTransform as TransformGroup;
                var translate = transformGroup.Children[1] as TranslateTransform;

                // Создаем анимацию прыжка
                DoubleAnimation bounceAnim = new DoubleAnimation();
                bounceAnim.To = -25; // Прыгаем вверх на 25 пикселей
                bounceAnim.Duration = TimeSpan.FromMilliseconds(250); // Скорость прыжка
                bounceAnim.AutoReverse = true; // Возвращаемся обратно
                bounceAnim.RepeatBehavior = new RepeatBehavior(2); // Прыгаем 2 раза
                bounceAnim.EasingFunction = new QuadraticEase { EasingMode = EasingMode.EaseOut }; // Физика гравитации

                // Запускаем анимацию прыжка
                translate.BeginAnimation(TranslateTransform.YProperty, bounceAnim);

                // Запускаем саму программу с небольшой задержкой (чтобы насладиться анимацией)
                await Task.Delay(300); 
                
                try
                {
                    Process.Start(new ProcessStartInfo(app.AppPath) { UseShellExecute = true });
                }
                catch { }
            }
        }

        // МЕНЮ: Закрыть док
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
