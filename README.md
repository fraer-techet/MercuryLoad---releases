using System;
using System.Collections.Generic;
using System.Diagnostics;
using System.Runtime.InteropServices;
using System.Windows;
using System.Windows.Interop;
using System.Windows.Media;

namespace MacDock
{
    // Класс, который описывает одно приложение в доке
    public class DockApp
    {
        public string Letter { get; set; }   // Буква, которая будет на иконке
        public string AppPath { get; set; }  // Путь к программе на компе
        public Brush Color { get; set; }     // Цвет иконки
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
            LoadApps(); // Загружаем наши приложения при старте
        }

        // Заполняем док программами
        private void LoadApps()
        {
            var myApps = new List<DockApp>
            {
                new DockApp { Letter = "E", AppPath = "explorer.exe", Color = new SolidColorBrush((Color)ColorConverter.ConvertFromString("#E2B022")) }, // Желтый проводник
                new DockApp { Letter = "W", AppPath = "msedge.exe", Color = new SolidColorBrush((Color)ColorConverter.ConvertFromString("#0078D7")) },   // Синий Edge
                new DockApp { Letter = "N", AppPath = "notepad.exe", Color = new SolidColorBrush((Color)ColorConverter.ConvertFromString("#4CAF50")) },  // Зеленый блокнот
                new DockApp { Letter = "C", AppPath = "calc.exe", Color = new SolidColorBrush((Color)ColorConverter.ConvertFromString("#607D8B")) }      // Серый калькулятор
            };

            // Привязываем список к дизайну
            AppList.ItemsSource = myApps;
        }

        // Метод, который срабатывает при клике на иконку
        private void AppIcon_Click(object sender, System.Windows.Input.MouseButtonEventArgs e)
        {
            // Узнаем, на какую именно иконку нажали
            var border = sender as System.Windows.FrameworkElement;
            var app = border.DataContext as DockApp;

            if (app != null)
            {
                try
                {
                    // Пытаемся запустить программу
                    Process.Start(new ProcessStartInfo(app.AppPath) { UseShellExecute = true });
                }
                catch (Exception ex)
                {
                    MessageBox.Show("Не удалось запустить: " + ex.Message);
                }
            }
        }

        private void Window_SourceInitialized(object sender, EventArgs e)
        {
            IntPtr hwnd = new WindowInteropHelper(this).Handle;
            int extendedStyle = GetWindowLong(hwnd, GWL_EXSTYLE);
            SetWindowLong(hwnd, GWL_EXSTYLE, extendedStyle | WS_EX_TOOLWINDOW | WS_EX_NOACTIVATE);
        }

        private void Window_Loaded(object sender, RoutedEventArgs e)
        {
            // Ставим окно по центру
            this.Left = (SystemParameters.PrimaryScreenWidth - this.Width) / 2;
            
            // Прижимаем в САМЫЙ низ экрана (на уровень панели задач)
            this.Top = SystemParameters.PrimaryScreenHeight - this.Height;
        }
    }
}
