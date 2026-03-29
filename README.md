using System;
using System.Windows;

namespace MacDock
{
    public partial class App : System.Windows.Application
    {
        // Этот метод запускается в самую первую миллисекунду старта программы
        protected override void OnStartup(StartupEventArgs e)
        {
            base.OnStartup(e);

            // 1. Ловим ошибки в интерфейсе (WPF)
            this.DispatcherUnhandledException += (sender, args) =>
            {
                MessageBox.Show("ОШИБКА ИНТЕРФЕЙСА:\n" + args.Exception.Message + "\n\nГде:\n" + args.Exception.StackTrace, "КРАШ!", MessageBoxButton.OK, MessageBoxImage.Error);
                args.Handled = true; // Пытаемся предотвратить закрытие
            };

            // 2. Ловим фатальные ошибки глубоко в системе и ядре
            AppDomain.CurrentDomain.UnhandledException += (sender, args) =>
            {
                Exception ex = (Exception)args.ExceptionObject;
                MessageBox.Show("ФАТАЛЬНЫЙ ВЫЛЕТ:\n" + ex.Message + "\n\nГде:\n" + ex.StackTrace, "КРАШ!", MessageBoxButton.OK, MessageBoxImage.Error);
            };
        }
    }
}
