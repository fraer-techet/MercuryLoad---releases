using System;
using System.Collections.Generic;
using System.Diagnostics;
using System.Runtime.InteropServices;
using System.Threading.Tasks;
using System.Windows;
using System.Windows.Controls;
using System.Windows.Interop;
using System.Windows.Media;
using System.Windows.Media.Animation;

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

        private async void AppIcon_Click(object sender, System.Windows.Input.MouseButtonEventArgs e)
        {
            var border = sender as FrameworkElement;
            var app = border.DataContext as DockApp;

            if (app != null && border != null)
            {
                // --- 1. УЛУЧШЕННЫЙ И ОСЛАБЛЕННЫЙ BOUNCE (ПРЫЖОК) ---
                var transformGroup = border.RenderTransform as TransformGroup;
                var translate = transformGroup.Children[1] as TranslateTransform;

                DoubleAnimation bounceAnim = new DoubleAnimation {
                    To = -8, // Прыжок стал очень маленьким (всего 8 пикселей)
                    Duration = TimeSpan.FromMilliseconds(120), // Очень быстрый
                    AutoReverse = true, // Сразу возвращается назад
                    RepeatBehavior = new RepeatBehavior(1), // Делает это только один раз
                    EasingFunction = new QuadraticEase { EasingMode = EasingMode.EaseOut }
                };
                translate.BeginAnimation(TranslateTransform.YProperty, bounceAnim);


                // --- 2. ИЛЛЮЗИЯ ОТКРЫТИЯ ПРИЛОЖЕНИЯ (ZOOM ИЗ ДОКА) ---
                
                // Вычисляем точную позицию иконки на экране монитора
                Point screenPoint = border.PointToScreen(new Point(0, 0));
                
                // Учитываем масштаб Windows (DPI), чтобы анимация была ровно на месте иконки
                PresentationSource source = PresentationSource.FromVisual(this);
                double dpiScale = source != null ? source.CompositionTarget.TransformToDevice.M11 : 1.0;
                
                // Создаем невидимое прозрачное окно на весь экран для анимации
                Window illusionWindow = new Window {
                    WindowStyle = WindowStyle.None, AllowsTransparency = true, Background = Brushes.Transparent,
                    Topmost = true, ShowInTaskbar = false, IsHitTestVisible = false,
                    Left = 0, Top = 0, Width = SystemParameters.PrimaryScreenWidth, Height = SystemParameters.PrimaryScreenHeight
                };

                Canvas canvas = new Canvas();
                illusionWindow.Content = canvas;

                // Создаем клона нашей иконки
                Border fakeIcon = new Border {
                    Width = border.ActualWidth, Height = border.ActualHeight,
                    Background = app.Color, CornerRadius = new CornerRadius(14),
                    Child = new TextBlock { 
                        Text = app.Letter, HorizontalAlignment = HorizontalAlignment.Center, 
                        VerticalAlignment = VerticalAlignment.Center, Foreground = Brushes.White, 
                        FontSize = 26, FontWeight = FontWeights.Bold 
                    }
                };

                // Размещаем клона ровно поверх настоящей иконки
                Canvas.SetLeft(fakeIcon, screenPoint.X / dpiScale);
                Canvas.SetTop(fakeIcon, screenPoint.Y / dpiScale);
                canvas.Children.Add(fakeIcon);

                // Настраиваем трансформацию для клона (увеличение из центра)
                ScaleTransform scaleTransform = new ScaleTransform(1, 1, border.ActualWidth / 2, border.ActualHeight / 2);
                fakeIcon.RenderTransform = scaleTransform;

                illusionWindow.Show();

                // Запускаем анимацию "Взрыва/Увеличения" клона иконки
                DoubleAnimation scaleUp = new DoubleAnimation(1, 15, TimeSpan.FromMilliseconds(300)) { 
                    EasingFunction = new QuarticEase { EasingMode = EasingMode.EaseIn } 
                };
                DoubleAnimation fadeOut = new DoubleAnimation(1, 0, TimeSpan.FromMilliseconds(300)) {
                    EasingFunction = new QuarticEase { EasingMode = EasingMode.EaseIn }
                };

                scaleTransform.BeginAnimation(ScaleTransform.ScaleXProperty, scaleUp);
                scaleTransform.BeginAnimation(ScaleTransform.ScaleYProperty, scaleUp);
                fakeIcon.BeginAnimation(UIElement.OpacityProperty, fadeOut);

                // Даем анимации чуть-чуть времени и запускаем настоящую программу
                await Task.Delay(100); 
                try {
                    Process.Start(new ProcessStartInfo(app.AppPath) { UseShellExecute = true });
                } catch { }

                // Ждем окончания анимации и удаляем невидимое окно
                await Task.Delay(250);
                illusionWindow.Close();
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
