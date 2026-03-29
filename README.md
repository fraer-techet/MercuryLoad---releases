<Window x:Class="MacDock.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="MacDock" Height="90" Width="800"
        WindowStyle="None" AllowsTransparency="True" Background="Transparent"
        Topmost="True" ShowInTaskbar="False"
        SourceInitialized="Window_SourceInitialized" Loaded="Window_Loaded">

    <Grid VerticalAlignment="Bottom">
        <!-- Красивая темная стеклянная подложка -->
        <Border Margin="10,0,10,5" CornerRadius="18" HorizontalAlignment="Center" VerticalAlignment="Bottom">
            <Border.Background>
                <!-- Темный полупрозрачный фон -->
                <SolidColorBrush Color="#CC151515"/> 
            </Border.Background>
            <!-- Тонкая белая окантовка сверху для объема -->
            <Border.BorderBrush>
                <LinearGradientBrush StartPoint="0,0" EndPoint="0,1">
                    <GradientStop Color="#40FFFFFF" Offset="0.0" />
                    <GradientStop Color="#00FFFFFF" Offset="1.0" />
                </LinearGradientBrush>
            </Border.BorderBrush>
            <Border.BorderThickness>1,1,1,0</Border.BorderThickness>
            
            <Border.Effect>
                <DropShadowEffect Color="Black" BlurRadius="20" ShadowDepth="5" Opacity="0.5" Direction="270"/>
            </Border.Effect>

            <!-- Список наших программ -->
            <ItemsControl x:Name="AppList" Margin="10,8,10,8">
                <!-- Располагаем их по горизонтали -->
                <ItemsControl.ItemsPanel>
                    <ItemsPanelTemplate>
                        <StackPanel Orientation="Horizontal" VerticalAlignment="Bottom"/>
                    </ItemsPanelTemplate>
                </ItemsControl.ItemsPanel>

                <!-- Дизайн и логика каждой отдельной иконки -->
                <ItemsControl.ItemTemplate>
                    <DataTemplate>
                        <!-- Кнопка приложения -->
                        <Border Width="50" Height="50" Margin="8,0,8,0" 
                                Background="{Binding Color}" CornerRadius="12" 
                                Cursor="Hand" MouseLeftButtonUp="AppIcon_Click">
                            
                            <!-- Точка трансформации для анимации (увеличение от нижнего края) -->
                            <Border.RenderTransform>
                                <ScaleTransform ScaleX="1" ScaleY="1" CenterX="25" CenterY="50" />
                            </Border.RenderTransform>

                            <!-- АНИМАЦИИ НАВЕДЕНИЯ МЫШИ -->
                            <Border.Style>
                                <Style TargetType="Border">
                                    <Style.Triggers>
                                        <!-- Когда мышь наведена -->
                                        <EventTrigger RoutedEvent="MouseEnter">
                                            <BeginStoryboard>
                                                <Storyboard>
                                                    <DoubleAnimation Storyboard.TargetProperty="RenderTransform.ScaleX" To="1.4" Duration="0:0:0.15" >
                                                        <DoubleAnimation.EasingFunction>
                                                            <QuadraticEase EasingMode="EaseOut"/>
                                                        </DoubleAnimation.EasingFunction>
                                                    </DoubleAnimation>
                                                    <DoubleAnimation Storyboard.TargetProperty="RenderTransform.ScaleY" To="1.4" Duration="0:0:0.15" >
                                                        <DoubleAnimation.EasingFunction>
                                                            <QuadraticEase EasingMode="EaseOut"/>
                                                        </DoubleAnimation.EasingFunction>
                                                    </DoubleAnimation>
                                                </Storyboard>
                                            </BeginStoryboard>
                                        </EventTrigger>
                                        <!-- Когда мышь ушла -->
                                        <EventTrigger RoutedEvent="MouseLeave">
                                            <BeginStoryboard>
                                                <Storyboard>
                                                    <DoubleAnimation Storyboard.TargetProperty="RenderTransform.ScaleX" To="1" Duration="0:0:0.15" />
                                                    <DoubleAnimation Storyboard.TargetProperty="RenderTransform.ScaleY" To="1" Duration="0:0:0.15" />
                                                </Storyboard>
                                            </BeginStoryboard>
                                        </EventTrigger>
                                    </Style.Triggers>
                                </Style>
                            </Border.Style>

                            <!-- Текст внутри иконки (пока картинок нет, используем первую букву программы) -->
                            <TextBlock Text="{Binding Letter}" HorizontalAlignment="Center" VerticalAlignment="Center" 
                                       Foreground="White" FontSize="24" FontWeight="Bold">
                                <TextBlock.Effect>
                                    <DropShadowEffect Color="Black" BlurRadius="2" ShadowDepth="1" Opacity="0.8"/>
                                </TextBlock.Effect>
                            </TextBlock>
                        </Border>
                    </DataTemplate>
                </ItemsControl.ItemTemplate>
            </ItemsControl>
        </Border>
    </Grid>
</Window>
