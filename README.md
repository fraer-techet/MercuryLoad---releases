                else 
                {
                    // ИСПРАВЛЕНИЕ: Создаем меню без страшного черного фона, чтобы оно подхватило наш стеклянный стиль из XAML
                    ContextMenu groupMenu = new ContextMenu();
                    
                    foreach(var win in app.OpenWindows)
                    {
                        MenuItem item = new MenuItem { Header = win.Title };
                        item.Click += (s, ev) => {
                            ShowWindow(win.Handle, SW_RESTORE);
                            SetForegroundWindow(win.Handle);
                        };
                        groupMenu.Items.Add(item);
                    }
                    
                    groupMenu.PlacementTarget = border;
                    groupMenu.Placement = System.Windows.Controls.Primitives.PlacementMode.Top;
                    groupMenu.IsOpen = true;
                }
