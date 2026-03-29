using System;
using System.Windows;

namespace MacDock
{
    public partial class App : System.Windows.Application
    {
        protected override void OnStartup(StartupEventArgs e)
        {
            base.OnStartup(e);

            this.DispatcherUnhandledException += (sender, args) =>
            {
                System.Windows.MessageBox.Show("ОШИБКА:\n" + args.Exception.Message, "КРАШ", System.Windows.MessageBoxButton.OK, System.Windows.MessageBoxImage.Error);
                args.Handled = true; 
            };

            AppDomain.CurrentDomain.UnhandledException += (sender, args) =>
            {
                Exception ex = (Exception)args.ExceptionObject;
                System.Windows.MessageBox.Show("ФАТАЛЬНЫЙ ВЫЛЕТ:\n" + ex.Message, "КРАШ", System.Windows.MessageBoxButton.OK, System.Windows.MessageBoxImage.Error);
            };
        }
    }
}
