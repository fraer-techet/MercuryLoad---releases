using System;
using System.Collections.Generic;
using System.Collections.ObjectModel;
using System.ComponentModel;
using System.Diagnostics;
using System.IO;
using System.Linq;
using System.Runtime.InteropServices;
using System.Text;
using System.Threading.Tasks;
using System.Windows;
using System.Windows.Controls;
using System.Windows.Interop;
using System.Windows.Media;
using System.Windows.Media.Imaging;
using System.Windows.Threading;

namespace MacDock
{
    public class WindowInfo
    {
        public IntPtr Handle { get; set; }
        public string Title { get; set; } = string.Empty;
    }

    public class DockApp : INotifyPropertyChanged
    {
        public string AppName { get; set; } = string.Empty;
        public string AppPath { get; set; } = string.Empty;
        public ImageSource? IconImage { get; set; }
        
        public List<WindowInfo> OpenWindows { get; set; } = new List<WindowInfo>();

        private double indicatorOpacity = 0.0;
        public double IndicatorOpacity { get => indicatorOpacity; set { indicatorOpacity = value; OnPropertyChanged(nameof(IndicatorOpacity)); } }

        private bool isPinned;
        public bool IsPinned { get => isPinned; set { isPinned = value; OnPropertyChanged(nameof(IsPinned)); OnPropertyChanged(nameof(PinText)); } }

        public string PinText => IsPinned ? "Открепить" : "Закрепить в доке";

        public event PropertyChangedEventHandler? PropertyChanged;
        protected void OnPropertyChanged(string name) => PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(name));
    }

    public partial class MainWindow : Window
    {
        [DllImport("user32.dll")] private static extern int GetWindowLong(IntPtr hwnd, int index);
        [DllImport("user32.dll")] private static extern int SetWindowLong(IntPtr hwnd, int index, int newStyle);
        [DllImport("user32.dll")] private static extern bool EnumWindows(EnumWindowsProc enumProc, IntPtr lParam);
        [DllImport("user32.dll")] private static extern bool IsWindowVisible(IntPtr hWnd);
        [DllImport("user32.dll", CharSet = CharSet.Auto)] private static extern int GetWindowTextLength(IntPtr hWnd);
        [DllImport("user32.dll", CharSet = CharSet.Auto)] private static extern int GetWindowText(IntPtr hWnd, StringBuilder lpString, int nMaxCount);
        [DllImport("user32.dll")] private static extern uint GetWindowThreadProcessId(IntPtr hWnd, out uint lpdwProcessId);
        [DllImport("user32.dll")] private static extern bool SetForegroundWindow(IntPtr hWnd);
        [DllImport("user32.dll")] private static extern bool ShowWindow(IntPtr hWnd, int nCmdShow);
        [DllImport("user32.dll")] private static extern IntPtr GetWindow(IntPtr hWnd, uint uCmd);
        [DllImport("user32.dll")] private static extern IntPtr GetForegroundWindow(); 
        [DllImport("dwmapi.dll")] private static extern int DwmGetWindowAttribute(IntPtr hwnd, int dwAttribute, out int pvAttribute, int cbAttribute);

        private delegate bool EnumWindowsProc(IntPtr hWnd, IntPtr lParam);

        private const int GWL_EXSTYLE = -20;
        private const int WS_EX_TOOLWINDOW = 0x00000080; 
        private const int WS_EX_APPWINDOW = 0x00040000;  
        private const int WS_EX_NOACTIVATE = 0x08000000;
        private const uint GW_OWNER = 4;
        private const int DWMWA_CLOAKED = 14;
        private const int SW_MINIMIZE = 6; 
        private const int SW_RESTORE = 9;  

        private ObservableCollection<DockApp> dockItems = new ObservableCollection<DockApp>();
        private DispatcherTimer scannerTimer = new DispatcherTimer();
        private string pinnedFilesPath = "pinned.txt"; 

        private bool isDragging = false;
        private Point dragStartPoint;
        private DockApp? draggedApp = null;
        private FrameworkElement? draggedElement = null;

        public MainWindow()
        {
            InitializeComponent();
            AppList.ItemsSource = dockItems;
            LoadPinnedApps(); 

            scannerTimer.Interval = TimeSpan.FromMilliseconds(500);
            scannerTimer.Tick += ScannerTimer_Tick;
            scannerTimer.Start();
        }

        private void ScannerTimer_Tick(object? sender, EventArgs e)
        {
            var groupedWindows = new Dictionary<string, List<WindowInfo>>();

            EnumWindows(delegate (IntPtr hWnd, IntPtr lParam)
            {
                if (IsTaskbarWindow(hWnd))
                {
                    GetWindowThreadProcessId(hWnd, out uint processId);
                    try
                    {
                        Process proc = Process.GetProcessById((int)processId);
                        string path = proc.MainModule?.FileName ?? "";

                        if (!string.IsNullOrEmpty(path) && !path.Contains("MacDock"))
                        {
                            StringBuilder title = new StringBuilder(GetWindowTextLength(hWnd) + 1);
                            GetWindowText(hWnd, title, title.Capacity);
                            
                            if (!groupedWindows.ContainsKey(path))
                                groupedWindows[path] = new List<WindowInfo>();

                            groupedWindows[path].Add(new WindowInfo { Handle = hWnd, Title = title.ToString() });
                        }
                    } catch { } 
                }
                return true;
            }, IntPtr.Zero);

            // Мгновенное удаление закрытых программ
            for (int i = dockItems.Count - 1; i >= 0; i--)
            {
                if (!groupedWindows.ContainsKey(dockItems[i].AppPath) && !dockItems[i].IsPinned)
                {
                    dockItems.RemoveAt(i);
                }
            }

            foreach (var app in dockItems)
            {
                if (groupedWindows.ContainsKey(app.AppPath))
                {
                    app.OpenWindows = groupedWindows[app.AppPath];
                    app.AppName = app.OpenWindows.Count == 1 ? app.OpenWindows[0].Title : $"{app.OpenWindows.Count} окон открыто";
                    app.IndicatorOpacity = 1.0; 
                }
                else
                {
                    app.OpenWindows.Clear();
                    app.IndicatorOpacity = 0.0;     
                }
            }

            foreach (var group in groupedWindows)
            {
                if (!dockItems.Any(a => a.AppPath == group.Key))
                {
                    dockItems.Add(new DockApp {
                        AppPath = group.Key, 
                        AppName = group.Value.Count == 1 ? group.Value[0].Title : $"{group.Value.Count} окон открыто", 
                        OpenWindows = group.Value,
                        IndicatorOpacity = 1.0, 
                        IsPinned = false, 
                        IconImage = GetIconFromFile(group.Key)
                    });
                }
            }
        }

        // ==========================================
        // ЛОГИКА DRAG & DROP И КЛИКОВ
        // ==========================================

        private void Icon_MouseDown(object sender, System.Windows.Input.MouseButtonEventArgs e)
        {
            draggedElement = sender as FrameworkElement;
            draggedApp = draggedElement?.DataContext as DockApp;
            dragStartPoint = e.GetPosition(this); 
            isDragging = false;
        }

        private void Icon_MouseMove(object sender, System.Windows.Input.MouseEventArgs e)
        {
            if (draggedApp == null || draggedElement == null || e.LeftButton != System.Windows.Input.MouseButtonState.Pressed)
                return;

            Point currentPoint = e.GetPosition(this);
            
            if (!isDragging && Math.Abs(currentPoint.X - dragStartPoint.X) > SystemParameters.MinimumHorizontalDragDistance)
            {
                isDragging = true;
                draggedElement.CaptureMouse(); 
            }

            if (isDragging)
            {
                Point posInList = e.GetPosition(AppList);
                int oldIndex = dockItems.IndexOf(draggedApp);
                int newIndex = GetTargetIndex(posInList);

                if (newIndex >= 0 && newIndex < dockItems.Count && oldIndex != newIndex)
                {
                    dockItems.Move(oldIndex, newIndex);
                }
            }
        }

        private int GetTargetIndex(Point mousePosInAppList)
        {
            double accumulatedWidth = 0;
            for (int i = 0; i < AppList.Items.Count; i++)
            {
                var container = AppList.ItemContainerGenerator.ContainerFromIndex(i) as FrameworkElement;
                if (container != null)
                {
                    accumulatedWidth += container.ActualWidth + container.Margin.Left + container.Margin.Right;
                    if (mousePosInAppList.X < accumulatedWidth) return i;
                }
            }
            return dockItems.Count - 1;
        }

        private void Icon_MouseUp(object sender, System.Windows.Input.MouseButtonEventArgs e)
        {
            if (isDragging)
            {
                draggedElement?.ReleaseMouseCapture();
                isDragging = false;
                SavePinnedApps(); 
            }
            else
            {
                if (draggedApp != null && draggedElement != null)
                {
                    ExecuteAppClick(draggedApp, draggedElement);
                }
            }

            draggedApp = null;
            draggedElement = null;
        }

        private async void ExecuteAppClick(DockApp app, FrameworkElement border)
        {
            var transformGroup = border.RenderTransform as TransformGroup;
            var translate = transformGroup?.Children[1] as TranslateTransform;
            if (translate != null)
            {
                System.Windows.Media.Animation.DoubleAnimation bounceAnim = new System.Windows.Media.Animation.DoubleAnimation {
                    To = -10, Duration = TimeSpan.FromMilliseconds(150), AutoReverse = true, RepeatBehavior = new System.Windows.Media.Animation.RepeatBehavior(1)
                };
                translate.BeginAnimation(TranslateTransform.YProperty, bounceAnim);
            }

            await Task.Delay(100);

            if (app.OpenWindows.Count == 0) 
            {
                try { Process.Start(new ProcessStartInfo(app.AppPath) { UseShellExecute = true }); } catch { }
            }
            else if (app.OpenWindows.Count == 1) 
            {
                IntPtr activeWindow = GetForegroundWindow();
                IntPtr myWindow = app.OpenWindows[0].Handle;

                if (activeWindow == myWindow) ShowWindow(myWindow, SW_MINIMIZE);
                else { ShowWindow(myWindow, SW_RESTORE); SetForegroundWindow(myWindow); }
            }
            else 
            {
                ContextMenu groupMenu = new ContextMenu { 
                    Background = new SolidColorBrush((System.Windows.Media.Color)System.Windows.Media.ColorConverter.ConvertFromString("#202020")), 
                    Foreground = System.Windows.Media.Brushes.White 
                };
                
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
        }

        private void PinApp_Click(object sender, RoutedEventArgs e)
        {
            var menuItem = sender as MenuItem;
            var app = menuItem?.DataContext as DockApp;
            if (app != null) { app.IsPinned = !app.IsPinned; SavePinnedApps(); }
        }

        private void SavePinnedApps()
        {
            var pinnedPaths = dockItems.Where(a => a.IsPinned).Select(a => a.AppPath);
            File.WriteAllLines(pinnedFilesPath, pinnedPaths);
        }

        private void LoadPinnedApps()
        {
            if (File.Exists(pinnedFilesPath))
            {
                foreach (var path in File.ReadAllLines(pinnedFilesPath))
                {
                    if (File.Exists(path) && !dockItems.Any(a => a.AppPath == path))
                    {
                        dockItems.Add(new DockApp {
                            AppPath = path, AppName = Path.GetFileNameWithoutExtension(path),
                            IsPinned = true, IndicatorOpacity = 0.0, IconImage = GetIconFromFile(path)
                        });
                    }
                }
            }
        }

        private bool IsTaskbarWindow(IntPtr hWnd)
        {
            if (!IsWindowVisible(hWnd)) return false;
            int titleLength = GetWindowTextLength(hWnd);
            if (titleLength == 0) return false;
            DwmGetWindowAttribute(hWnd, DWMWA_CLOAKED, out int cloaked, sizeof(int));
            if (cloaked != 0) return false;
            int exStyle = GetWindowLong(hWnd, GWL_EXSTYLE);
            IntPtr owner = GetWindow(hWnd, GW_OWNER);
            if ((exStyle & WS_EX_TOOLWINDOW) != 0 && (exStyle & WS_EX_APPWINDOW) == 0) return false;
            if (owner != IntPtr.Zero && (exStyle & WS_EX_APPWINDOW) == 0) return false;
            StringBuilder title = new StringBuilder(titleLength + 1);
            GetWindowText(hWnd, title, title.Capacity);
            string windowTitle = title.ToString();
            if (windowTitle == "Program Manager" || windowTitle == "Settings") return false;
            return true; 
        }

        private ImageSource? GetIconFromFile(string filePath)
        {
            try
            {
                using (System.Drawing.Icon? sysIcon = System.Drawing.Icon.ExtractAssociatedIcon(filePath))
                {
                    if (sysIcon != null)
                        return Imaging.CreateBitmapSourceFromHIcon(sysIcon.Handle, Int32Rect.Empty, BitmapSizeOptions.FromEmptyOptions());
                }
            } catch { } return null;
        }

        private void ExitMenu_Click(object sender, RoutedEventArgs e) => System.Windows.Application.Current.Shutdown();

        private void Window_SourceInitialized(object? sender, EventArgs e)
        {
            IntPtr hwnd = new WindowInteropHelper(this).Handle;
            SetWindowLong(hwnd, GWL_EXSTYLE, GetWindowLong(hwnd, GWL_EXSTYLE) | WS_EX_TOOLWINDOW | WS_EX_NOACTIVATE);
        }

        private void Window_Loaded(object sender, RoutedEventArgs e)
        {
            this.Left = (SystemParameters.PrimaryScreenWidth - this.Width) / 2;
            this.Top = SystemParameters.PrimaryScreenHeight - this.Height;
        }
    }
}
