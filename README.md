<Window x:Class="MacDock.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="MacDock" Height="110" Width="800"
        WindowStyle="None" AllowsTransparency="True" Background="Transparent"
        Topmost="True" ShowInTaskbar="False"
        SourceInitialized="Window_SourceInitialized" Loaded="Window_Loaded">

    <Window.ContextMenu>
        <ContextMenu Background="#1A1A1A" Foreground="White" BorderBrush="#33FFFFFF">
            <MenuItem Header="Настройки дока (Скоро)" IsEnabled="False"/>
            <Separator Background="#33FFFFFF"/>
            <MenuItem Header="Выйти" Click="ExitMenu_Click" />
        </ContextMenu>
    </Window.ContextMenu>

    <Grid VerticalAlignment="Bottom">
        <!-- Тот самый красивый стеклянный док -->
        <Border Margin="10,0,10,10" CornerRadius="22" HorizontalAlignment="Center" VerticalAlignment="Bottom"
                Background="#D9151515" BorderThickness="1,1,1,0">
            <Border.BorderBrush>
                <LinearGradientBrush StartPoint="0,0" EndPoint="0,1">
                    <GradientStop Color="#40FFFFFF" Offset="0.0" />
                    <GradientStop Color="#00FFFFFF" Offset="1.0" />
                </LinearGradientBrush>
            </Border.BorderBrush>
            <Border.Effect>
                <DropShadowEffect Color="Black" BlurRadius="30" ShadowDepth="10" Opacity="0.5" Direction="270"/>
            </Border.Effect>

            <ItemsControl x:Name="AppList" Margin="15,10,15,10">
                <ItemsControl.ItemsPanel>
                    <ItemsPanelTemplate>
                        <!-- Панель, где лежат иконки -->
                        <StackPanel Orientation="Horizontal" VerticalAlignment="Bottom"/>
                    </ItemsPanelTemplate>
                </ItemsControl.ItemsPanel>

                <ItemsControl.ItemTemplate>
                    <DataTemplate>
                        <!-- Контейнер для НАСТОЯЩЕЙ иконки -->
                        <Border Width="50" Height="50" Margin="8,0,8,0" 
                                Cursor="Hand" MouseLeftButtonUp="AppIcon_Click"
                                Background="Transparent"
                                ToolTip="{Binding AppName}">
                            
                            <!-- Точка роста: Центр по ширине (0.5), Самый низ по высоте (1) -->
                            <Border.RenderTransformOrigin>
                                <Point X="0.5" Y="1"/>
                            </Border.RenderTransformOrigin>

                            <!-- Трансформации: 0 - Масштаб, 1 - Прыжок -->
                            <Border.RenderTransform>
                                <TransformGroup>
                                    <ScaleTransform ScaleX="1" ScaleY="1" />
                                    <TranslateTransform Y="0" />
                                </TransformGroup>
                            </Border.RenderTransform>

                            <!-- АНИМАЦИИ НАВЕДЕНИЯ (ИДЕАЛЬНАЯ ПЛАВНОСТЬ) -->
                            <Border.Style>
                                <Style TargetType="Border">
                                    <Setter Property="Panel.ZIndex" Value="0"/>
                                    <Style.Triggers>
                                        
                                        <!-- ВОТ ИСПРАВЛЕНИЕ: Вынесли ZIndex в отдельный правильный триггер -->
                                        <Trigger Property="IsMouseOver" Value="True">
                                            <Setter Property="Panel.ZIndex" Value="100"/>
                                        </Trigger>

                                        <EventTrigger RoutedEvent="MouseEnter">
                                            <BeginStoryboard>
                                                <Storyboard>
                                                    <DoubleAnimation Storyboard.TargetProperty="(UIElement.RenderTransform).(TransformGroup.Children)[0].(ScaleTransform.ScaleX)" To="1.6" Duration="0:0:0.15">
                                                        <DoubleAnimation.EasingFunction><CubicEase EasingMode="EaseOut"/></DoubleAnimation.EasingFunction>
                                                    </DoubleAnimation>
                                                    <DoubleAnimation Storyboard.TargetProperty="(UIElement.RenderTransform).(TransformGroup.Children)[0].(ScaleTransform.ScaleY)" To="1.6" Duration="0:0:0.15">
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

                            <!-- САМА КАРТИНКА ПРИЛОЖЕНИЯ -->
                            <Image Source="{Binding IconImage}" Stretch="Uniform">
                                <Image.Effect>
                                    <DropShadowEffect Color="Black" BlurRadius="5" ShadowDepth="2" Opacity="0.4"/>
                                </Image.Effect>
                            </Image>
                        </Border>
                    </DataTemplate>
                </ItemsControl.ItemTemplate>
            </ItemsControl>
        </Border>
    </Grid>
</Window>
