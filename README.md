using System;
using System.IO;
using System.Windows;

namespace MacDock
{
    public partial class SettingsWindow : Window
    {
        private string configPath = "config.ini";

        public SettingsWindow()
        {
            InitializeComponent();
            LoadCurrentSettings();
        }

        private void LoadCurrentSettings()
        {
            if (File.Exists(configPath))
            {
                var lines = File.ReadAllLines(configPath);
                if (lines.Length >= 3)
                {
                    SizeSlider.Value = double.Parse(lines[0]);
                    OpacitySlider.Value = double.Parse(lines[1]);
                    MarginSlider.Value = double.Parse(lines[2]);
                }
            }
        }

        private void Save_Click(object sender, RoutedEventArgs e)
        {
            string[] settings = {
                SizeSlider.Value.ToString(),
                OpacitySlider.Value.ToString(),
                MarginSlider.Value.ToString()
            };
            File.WriteAllLines(configPath, settings);

            // Перезапуск
            System.Diagnostics.Process.Start(Environment.ProcessPath!);
            Application.Current.Shutdown();
        }
    }
}
