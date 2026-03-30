<Window x:Class="MacDock.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="MacDock" Height="90" Width="1000"
        WindowStyle="None" AllowsTransparency="True" Background="Transparent"
        Topmost="True" ShowInTaskbar="False"
        SourceInitialized="Window_SourceInitialized" Loaded="Window_Loaded">

    <Window.ContextMenu>
        <ContextMenu Background="#1A1A1A" Foreground="White" BorderBrush="#33FFFFFF">
            <MenuItem Header="⚙ Настройки Дока" Click="SettingsMenu_Click"/>
            <Separator Background="#33FFFFFF"/>
            <MenuItem Header="Выйти из дока" Click="ExitMenu_Click" />
        </ContextMenu>
    </Window.ContextMenu>

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
                        <!-- ИСПРАВЛЕНИЕ: Убрали ClipToBounds и добавили Background="Transparent", чтобы мышь ловилась идеально -->
                        <Grid VerticalAlignment="Bottom" Background="Transparent">
                            
                            <Grid.Style>
                                <Style TargetType="Grid">
                                    <Setter Property="Width" Value="52"/>
                                    <Setter Property="Opacity" Value="1"/>
                                    
                                    <!-- ИСПРАВЛЕНИЕ: Базовый слой 0 -->
                                    <Setter Property="Panel.ZIndex" Value="0"/>
                                    
                                    <Style.Triggers>
                                        <!-- ИСПРАВЛЕНИЕ: Поднимаем иконку поверх всех соседей при наведении! -->
                                        <Trigger Property="IsMouseOver" Value="True">
                                            <Setter Property="Panel.ZIndex" Value="100"/>
                                        </Trigger>
                                        
                                        <!-- Эффект "Призрака" при перетаскивании (сверх-слой) -->
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
                                    
                                    <!-- Контекстное меню -->
                                    <Border.ContextMenu>
                                        <ContextMenu Background="#202020" Foreground="White" BorderBrush="#40FFFFFF">
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
                                                <!-- Анимация увеличения (теперь без обрезания краев!) -->
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
