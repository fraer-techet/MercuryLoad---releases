<Window x:Class="MacDock.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="MacDock" Height="100" Width="800"
        WindowStyle="None" AllowsTransparency="True" Background="Transparent"
        Topmost="True" ShowInTaskbar="False"
        SourceInitialized="Window_SourceInitialized" Loaded="Window_Loaded">

    <!-- Добавляем меню по правому клику для закрытия дока -->
    <Window.ContextMenu>
        <ContextMenu Background="#202020" Foreground="White" BorderBrush="#40FFFFFF">
            <MenuItem Header="Выйти из дока" Click="ExitMenu_Click" />
        </ContextMenu>
    </Window.ContextMenu>

    <Grid VerticalAlignment="Bottom">
        <Border Margin="10,0,10,10" CornerRadius="20" HorizontalAlignment="Center" VerticalAlignment="Bottom">
            <Border.Background>
                <SolidColorBrush Color="#D9151515"/> 
            </Border.Background>
            <Border.BorderBrush>
                <LinearGradientBrush StartPoint="0,0" EndPoint="0,1">
                    <GradientStop Color="#50FFFFFF" Offset="0.0" />
                    <GradientStop Color="#00FFFFFF" Offset="1.0" />
                </LinearGradientBrush>
            </Border.BorderBrush>
            <Border.BorderThickness>1,1,1,0</Border.BorderThickness>
            
            <Border.Effect>
                <DropShadowEffect Color="Black" BlurRadius="25" ShadowDepth="10" Opacity="0.6" Direction="270"/>
            </Border.Effect>

            <ItemsControl x:Name="AppList" Margin="12,10,12,10">
                <ItemsControl.ItemsPanel>
                    <ItemsPanelTemplate>
                        <StackPanel Orientation="Horizontal" VerticalAlignment="Bottom"/>
                    </ItemsPanelTemplate>
                </ItemsControl.ItemsPanel>

                <ItemsControl.ItemTemplate>
                    <DataTemplate>
                        <Border Width="50" Height="50" Margin="8,0,8,0" 
                                Background="{Binding Color}" CornerRadius="14" 
                                Cursor="Hand" MouseLeftButtonUp="AppIcon_Click">
                            
                            <!-- ТЕПЕРЬ ТУТ ГРУППА ТРАНСФОРМАЦИЙ (Увеличение + Подпрыгивание) -->
                            <Border.RenderTransform>
                                <TransformGroup>
                                    <ScaleTransform ScaleX="1" ScaleY="1" CenterX="25" CenterY="50" />
                                    <TranslateTransform Y="0" />
                                </TransformGroup>
                            </Border.RenderTransform>

                            <!-- УЛУЧШЕННЫЕ АНИМАЦИИ НАВЕДЕНИЯ -->
                            <Border.Style>
                                <Style TargetType="Border">
                                    <Style.Triggers>
                                        <EventTrigger RoutedEvent="MouseEnter">
                                            <BeginStoryboard>
                                                <Storyboard>
                                                    <!-- CubicEase делает анимацию маслянистой -->
                                                    <DoubleAnimation Storyboard.TargetProperty="(UIElement.RenderTransform).(TransformGroup.Children)[0].(ScaleTransform.ScaleX)" To="1.5" Duration="0:0:0.2">
                                                        <DoubleAnimation.EasingFunction>
                                                            <CubicEase EasingMode="EaseOut"/>
                                                        </DoubleAnimation.EasingFunction>
                                                    </DoubleAnimation>
                                                    <DoubleAnimation Storyboard.TargetProperty="(UIElement.RenderTransform).(TransformGroup.Children)[0].(ScaleTransform.ScaleY)" To="1.5" Duration="0:0:0.2">
                                                        <DoubleAnimation.EasingFunction>
                                                            <CubicEase EasingMode="EaseOut"/>
                                                        </DoubleAnimation.EasingFunction>
                                                    </DoubleAnimation>
                                                </Storyboard>
                                            </BeginStoryboard>
                                        </EventTrigger>
                                        <EventTrigger RoutedEvent="MouseLeave">
                                            <BeginStoryboard>
                                                <Storyboard>
                                                    <DoubleAnimation Storyboard.TargetProperty="(UIElement.RenderTransform).(TransformGroup.Children)[0].(ScaleTransform.ScaleX)" To="1" Duration="0:0:0.25">
                                                        <DoubleAnimation.EasingFunction>
                                                            <CubicEase EasingMode="EaseOut"/>
                                                        </DoubleAnimation.EasingFunction>
                                                    </DoubleAnimation>
                                                    <DoubleAnimation Storyboard.TargetProperty="(UIElement.RenderTransform).(TransformGroup.Children)[0].(ScaleTransform.ScaleY)" To="1" Duration="0:0:0.25">
                                                        <DoubleAnimation.EasingFunction>
                                                            <CubicEase EasingMode="EaseOut"/>
                                                        </DoubleAnimation.EasingFunction>
                                                    </DoubleAnimation>
                                                </Storyboard>
                                            </BeginStoryboard>
                                        </EventTrigger>
                                    </Style.Triggers>
                                </Style>
                            </Border.Style>

                            <TextBlock Text="{Binding Letter}" HorizontalAlignment="Center" VerticalAlignment="Center" 
                                       Foreground="White" FontSize="26" FontWeight="Bold">
                                <TextBlock.Effect>
                                    <DropShadowEffect Color="Black" BlurRadius="4" ShadowDepth="2" Opacity="0.5"/>
                                </TextBlock.Effect>
                            </TextBlock>
                        </Border>
                    </DataTemplate>
                </ItemsControl.ItemTemplate>
            </ItemsControl>
        </Border>
    </Grid>
</Window>
