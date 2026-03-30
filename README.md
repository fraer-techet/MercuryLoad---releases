<Window x:Class="MacDock.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="MacDock" Height="90" Width="1000"
        WindowStyle="None" AllowsTransparency="True" Background="Transparent"
        Topmost="True" ShowInTaskbar="False"
        SourceInitialized="Window_SourceInitialized" Loaded="Window_Loaded">

    <!-- ГЛОБАЛЬНЫЕ СТИЛИ ДЛЯ КРАСИВЫХ МЕНЮШЕК -->
    <Window.Resources>
        <!-- Стиль для самого окошка меню -->
        <Style TargetType="ContextMenu">
            <Setter Property="Template">
                <Setter.Value>
                    <ControlTemplate TargetType="ContextMenu">
                        <!-- Стеклянная подложка меню -->
                        <Border Background="#D9151515" 
                                BorderBrush="#30FFFFFF" BorderThickness="1" 
                                CornerRadius="12" Margin="10">
                            <!-- Тень от меню -->
                            <Border.Effect>
                                <DropShadowEffect Color="Black" BlurRadius="15" ShadowDepth="5" Opacity="0.5"/>
                            </Border.Effect>
                            <!-- Список кнопок внутри -->
                            <StackPanel IsItemsHost="True" Margin="5" KeyboardNavigation.DirectionalNavigation="Cycle"/>
                        </Border>
                    </ControlTemplate>
                </Setter.Value>
            </Setter>
        </Style>

        <!-- Стиль для каждой отдельной кнопки в меню -->
        <Style TargetType="MenuItem">
            <Setter Property="Foreground" Value="White"/>
            <Setter Property="FontSize" Value="14"/>
            <Setter Property="FontFamily" Value="Segoe UI"/>
            <Setter Property="Padding" Value="15,8,15,8"/>
            <Setter Property="Template">
                <Setter.Value>
                    <ControlTemplate TargetType="MenuItem">
                        <Border x:Name="MenuBorder" Background="Transparent" CornerRadius="6" Margin="0,2,0,2">
                            <ContentPresenter ContentSource="Header" Margin="{TemplateBinding Padding}" />
                        </Border>
                        <ControlTemplate.Triggers>
                            <!-- При наведении кнопка становится светло-серой и закругленной -->
                            <Trigger Property="IsHighlighted" Value="True">
                                <Setter TargetName="MenuBorder" Property="Background" Value="#40FFFFFF"/>
                            </Trigger>
                            <!-- Если кнопка отключена (серая) -->
                            <Trigger Property="IsEnabled" Value="False">
                                <Setter Property="Foreground" Value="#555555"/>
                            </Trigger>
                        </ControlTemplate.Triggers>
                    </ControlTemplate>
                </Setter.Value>
            </Setter>
        </Style>

        <!-- Стиль для разделителя в меню -->
        <Style TargetType="Separator">
            <Setter Property="Template">
                <Setter.Value>
                    <ControlTemplate TargetType="Separator">
                        <Border Height="1" Background="#20FFFFFF" Margin="10,5,10,5"/>
                    </ControlTemplate>
                </Setter.Value>
            </Setter>
        </Style>
    </Window.Resources>

    <!-- Главное меню (правый клик по доку) -->
    <Window.ContextMenu>
        <ContextMenu>
            <MenuItem Header="⚙ Настройки Дока" Click="SettingsMenu_Click"/>
            <Separator/>
            <MenuItem Header="Выйти из MacDock" Click="ExitMenu_Click" />
        </ContextMenu>
    </Window.ContextMenu>

    <!-- САМ ДОК -->
    <Grid VerticalAlignment="Bottom">
        <Border x:Name="DockBackground" Margin="10,0,10,0" CornerRadius="18" HorizontalAlignment="Center" VerticalAlignment="Bottom"
                Background="#D9151515" BorderThickness="1,1,1,0">
            <Border.BorderBrush>
                <LinearGradientBrush StartPoint="0,0" EndPoint="0,1">
                    <GradientStop Color="#40FFFFFF" Offset="0.0" />
                    <GradientStop Color="#00FFFFFF" Offset="1.0" />
                </LinearGradientBrush>
            </Border.BorderBrush>
            <Border.Effect>
                <DropShadowEffect Color="Black" BlurRadius="25" ShadowDepth="8" Opacity="0.5" Direction="270"/>
            </Border.Effect>

            <ItemsControl x:Name="AppList" Margin="12,8,12,6">
                <!-- Система масштабирования из настроек -->
                <ItemsControl.LayoutTransform>
                    <ScaleTransform x:Name="DockScale" ScaleX="1" ScaleY="1" />
                </ItemsControl.LayoutTransform>
                
                <ItemsControl.ItemsPanel>
                    <ItemsPanelTemplate>
                        <StackPanel Orientation="Horizontal" VerticalAlignment="Bottom"/>
                    </ItemsPanelTemplate>
                </ItemsControl.ItemsPanel>

                <ItemsControl.ItemTemplate>
                    <DataTemplate>
                        <Grid VerticalAlignment="Bottom" Background="Transparent">
                            
                            <Grid.Style>
                                <Style TargetType="Grid">
                                    <Setter Property="Width" Value="52"/>
                                    <Setter Property="Opacity" Value="1"/>
                                    <Setter Property="Panel.ZIndex" Value="0"/>
                                    
                                    <Style.Triggers>
                                        <Trigger Property="IsMouseOver" Value="True">
                                            <Setter Property="Panel.ZIndex" Value="100"/>
                                        </Trigger>
                                        <DataTrigger Binding="{Binding IsBeingDragged}" Value="True">
                                            <Setter Property="Panel.ZIndex" Value="999"/>
                                            <Setter Property="Opacity" Value="0.6"/>
                                        </DataTrigger>
                                        <EventTrigger RoutedEvent="Loaded">
                                            <BeginStoryboard>
                                                <Storyboard>
                                                    <DoubleAnimation Storyboard.TargetProperty="Width" From="0" To="52" Duration="0:0:0.25">
                                                        <DoubleAnimation.EasingFunction><CubicEase EasingMode="EaseOut"/></DoubleAnimation.EasingFunction>
                                                    </DoubleAnimation>
                                                    <DoubleAnimation Storyboard.TargetProperty="Opacity" From="0" To="1" Duration="0:0:0.25"/>
                                                </Storyboard>
                                            </BeginStoryboard>
                                        </EventTrigger>
                                        <DataTrigger Binding="{Binding IsClosing}" Value="True">
                                            <DataTrigger.EnterActions>
                                                <BeginStoryboard x:Name="CloseAnim">
                                                    <Storyboard>
                                                        <DoubleAnimation Storyboard.TargetProperty="Width" To="0" Duration="0:0:0.25">
                                                            <DoubleAnimation.EasingFunction><CubicEase EasingMode="EaseOut"/></DoubleAnimation.EasingFunction>
                                                        </DoubleAnimation>
                                                        <DoubleAnimation Storyboard.TargetProperty="Opacity" To="0" Duration="0:0:0.25"/>
                                                    </Storyboard>
                                                </BeginStoryboard>
                                            </DataTrigger.EnterActions>
                                            <DataTrigger.ExitActions>
                                                <StopStoryboard BeginStoryboardName="CloseAnim"/>
                                            </DataTrigger.ExitActions>
                                        </DataTrigger>
                                    </Style.Triggers>
                                </Style>
                            </Grid.Style>

                            <StackPanel HorizontalAlignment="Center" VerticalAlignment="Bottom">
                                <Border Width="40" Height="40" Cursor="Hand" 
                                        PreviewMouseLeftButtonDown="Icon_MouseDown" 
                                        PreviewMouseMove="Icon_MouseMove" 
                                        PreviewMouseLeftButtonUp="Icon_MouseUp"
                                        Background="Transparent" ToolTip="{Binding AppName}">
                                    
                                    <!-- Меню на иконке (Оно автоматически подхватит красивый стиль!) -->
                                    <Border.ContextMenu>
                                        <ContextMenu>
                                            <MenuItem Header="{Binding PinText}" Click="PinApp_Click"/>
                                        </ContextMenu>
                                    </Border.ContextMenu>

                                    <Border.RenderTransformOrigin>
                                        <Point X="0.5" Y="1"/>
                                    </Border.RenderTransformOrigin>

                                    <Border.RenderTransform>
                                        <TransformGroup>
                                            <ScaleTransform ScaleX="1" ScaleY="1" />
                                            <TranslateTransform Y="0" />
                                        </TransformGroup>
                                    </Border.RenderTransform>

                                    <Border.Style>
                                        <Style TargetType="Border">
                                            <Style.Triggers>
                                                <EventTrigger RoutedEvent="MouseEnter">
                                                    <BeginStoryboard>
                                                        <Storyboard>
                                                            <DoubleAnimation Storyboard.TargetProperty="(UIElement.RenderTransform).(TransformGroup.Children)[0].(ScaleTransform.ScaleX)" To="1.5" Duration="0:0:0.15">
                                                                <DoubleAnimation.EasingFunction><CubicEase EasingMode="EaseOut"/></DoubleAnimation.EasingFunction>
                                                            </DoubleAnimation>
                                                            <DoubleAnimation Storyboard.TargetProperty="(UIElement.RenderTransform).(TransformGroup.Children)[0].(ScaleTransform.ScaleY)" To="1.5" Duration="0:0:0.15">
                                                                <DoubleAnimation.EasingFunction><CubicEase EasingMode="EaseOut"/></DoubleAnimation.EasingFunction>
                                                            </DoubleAnimation>
                                                        </Storyboard>
                                                    </BeginStoryboard>
                                                </EventTrigger>
                                                <EventTrigger RoutedEvent="MouseLeave">
                                                    <BeginStoryboard>
                                                        <Storyboard>
                                                            <DoubleAnimation Storyboard.TargetProperty="(UIElement.RenderTransform).(TransformGroup.Children)[0].(ScaleTransform.ScaleX)" To="1" Duration="0:0:0.25">
                                                                <DoubleAnimation.EasingFunction><CubicEase EasingMode="EaseOut"/></DoubleAnimation.EasingFunction>
                                                            </DoubleAnimation>
                                                            <DoubleAnimation Storyboard.TargetProperty="(UIElement.RenderTransform).(TransformGroup.Children)[0].(ScaleTransform.ScaleY)" To="1" Duration="0:0:0.25">
                                                                <DoubleAnimation.EasingFunction><CubicEase EasingMode="EaseOut"/></DoubleAnimation.EasingFunction>
                                                            </DoubleAnimation>
                                                        </Storyboard>
                                                    </BeginStoryboard>
                                                </EventTrigger>
                                            </Style.Triggers>
                                        </Style>
                                    </Border.Style>

                                    <Image Source="{Binding IconImage}" Stretch="Uniform">
                                        <Image.Effect>
                                            <DropShadowEffect Color="Black" BlurRadius="4" ShadowDepth="2" Opacity="0.4"/>
                                        </Image.Effect>
                                    </Image>
                                </Border>
                                
                                <Ellipse Width="4" Height="4" Fill="White" Margin="0,4,0,0" Opacity="{Binding IndicatorOpacity}">
                                    <Ellipse.Effect>
                                        <DropShadowEffect Color="White" BlurRadius="5" ShadowDepth="0"/>
                                    </Ellipse.Effect>
                                </Ellipse>
                            </StackPanel>
                        </Grid>
                    </DataTemplate>
                </ItemsControl.ItemTemplate>
            </ItemsControl>
        </Border>
    </Grid>
</Window>
