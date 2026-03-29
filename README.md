<Window x:Class="MacDock.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="MacDock" Height="90" Width="800"
        WindowStyle="None" AllowsTransparency="True" Background="Transparent"
        Topmost="True" ShowInTaskbar="False"
        SourceInitialized="Window_SourceInitialized" Loaded="Window_Loaded">

    <Window.ContextMenu>
        <ContextMenu Background="#1A1A1A" Foreground="White" BorderBrush="#33FFFFFF">
            <MenuItem Header="Настройки дизайна (Скоро)" IsEnabled="False"/>
            <Separator Background="#33FFFFFF"/>
            <MenuItem Header="Выйти из дока" Click="ExitMenu_Click" />
        </ContextMenu>
    </Window.ContextMenu>

    <Grid VerticalAlignment="Bottom">
        <Border Margin="10,0,10,10" CornerRadius="18" HorizontalAlignment="Center" VerticalAlignment="Bottom"
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
                <ItemsControl.ItemsPanel>
                    <ItemsPanelTemplate>
                        <StackPanel Orientation="Horizontal" VerticalAlignment="Bottom"/>
                    </ItemsPanelTemplate>
                </ItemsControl.ItemsPanel>

                <ItemsControl.ItemTemplate>
                    <DataTemplate>
                        <!-- Обертка для анимации появления и ИСЧЕЗНОВЕНИЯ -->
                        <Grid VerticalAlignment="Bottom">
                            
                            <!-- Точка сжатия (уменьшается по горизонтали) -->
                            <Grid.LayoutTransform>
                                <ScaleTransform ScaleX="0" ScaleY="1"/>
                            </Grid.LayoutTransform>
                            
                            <Grid.Style>
                                <Style TargetType="Grid">
                                    <Style.Triggers>
                                        <!-- ПЛАВНОЕ ПОЯВЛЕНИЕ -->
                                        <EventTrigger RoutedEvent="Loaded">
                                            <BeginStoryboard>
                                                <Storyboard>
                                                    <DoubleAnimation Storyboard.TargetProperty="(UIElement.LayoutTransform).(ScaleTransform.ScaleX)" From="0" To="1" Duration="0:0:0.25">
                                                        <DoubleAnimation.EasingFunction><CubicEase EasingMode="EaseOut"/></DoubleAnimation.EasingFunction>
                                                    </DoubleAnimation>
                                                    <DoubleAnimation Storyboard.TargetProperty="Opacity" From="0" To="1" Duration="0:0:0.25"/>
                                                </Storyboard>
                                            </BeginStoryboard>
                                        </EventTrigger>

                                        <!-- ПЛАВНОЕ ИСЧЕЗНОВЕНИЕ (СУЖЕНИЕ) -->
                                        <DataTrigger Binding="{Binding IsClosing}" Value="True">
                                            <DataTrigger.EnterActions>
                                                <BeginStoryboard x:Name="CloseAnim">
                                                    <Storyboard>
                                                        <DoubleAnimation Storyboard.TargetProperty="(UIElement.LayoutTransform).(ScaleTransform.ScaleX)" To="0" Duration="0:0:0.25">
                                                            <DoubleAnimation.EasingFunction><CubicEase EasingMode="EaseOut"/></DoubleAnimation.EasingFunction>
                                                        </DoubleAnimation>
                                                        <DoubleAnimation Storyboard.TargetProperty="Opacity" To="0" Duration="0:0:0.25"/>
                                                    </Storyboard>
                                                </BeginStoryboard>
                                            </DataTrigger.EnterActions>
                                            <!-- Если ты резко передумал и снова открыл прогу пока она сжималась - она "оживет" -->
                                            <DataTrigger.ExitActions>
                                                <StopStoryboard BeginStoryboardName="CloseAnim"/>
                                                <BeginStoryboard>
                                                    <Storyboard>
                                                        <DoubleAnimation Storyboard.TargetProperty="(UIElement.LayoutTransform).(ScaleTransform.ScaleX)" To="1" Duration="0:0:0.2"/>
                                                        <DoubleAnimation Storyboard.TargetProperty="Opacity" To="1" Duration="0:0:0.2"/>
                                                    </Storyboard>
                                                </BeginStoryboard>
                                            </DataTrigger.ExitActions>
                                        </DataTrigger>
                                    </Style.Triggers>
                                </Style>
                            </Grid.Style>

                            <StackPanel Margin="6,0,6,0" VerticalAlignment="Bottom">
                                <Border Width="40" Height="40" Cursor="Hand" MouseLeftButtonUp="AppIcon_Click"
                                        Background="Transparent" ToolTip="{Binding AppName}">
                                    
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
                                            <Setter Property="Panel.ZIndex" Value="0"/>
                                            <Style.Triggers>
                                                <Trigger Property="IsMouseOver" Value="True">
                                                    <Setter Property="Panel.ZIndex" Value="100"/>
                                                </Trigger>
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
