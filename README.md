        // --- МЕТОД ЗАДЕРЖКИ ПЕРЕД УДАЛЕНИЕМ (С ЗАЩИТОЙ ОТ ВЫЛЕТОВ) ---
        private async void RemoveAppDelayed(DockApp app)
        {
            await Task.Delay(250); // Ждем ровно столько, сколько длится анимация
            
            if (app.IsClosing) 
            {
                // Заставляем систему удалять элемент ТОЛЬКО в безопасном потоке интерфейса
                Application.Current.Dispatcher.Invoke(() => {
                    if (dockItems.Contains(app))
                    {
                        dockItems.Remove(app);
                    }
                });
            }
        }
