<Window x:Class="MacDock.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="MacDock" Height="90" Width="600"
        WindowStyle="None" 
        AllowsTransparency="True" 
        Background="Transparent"
        Topmost="True" 
        ShowInTaskbar="False"
        SourceInitialized="Window_SourceInitialized"
        Loaded="Window_Loaded">
    
    <Grid>
        <!-- Стеклянная подложка дока -->
        <Border Margin="10,10,10,15" CornerRadius="20">
            <!-- Сложный градиент для эффекта матового стекла (как в macOS) -->
            <Border.Background>
                <LinearGradientBrush StartPoint="0,0" EndPoint="0,1">
                    <GradientStop Color="#50FFFFFF" Offset="0.0" />
                    <GradientStop Color="#30FFFFFF" Offset="1.0" />
                </LinearGradientBrush>
            </Border.Background>
            
            <!-- Рамка с бликом (верхняя грань чуть светлее) -->
            <Border.BorderBrush>
                <LinearGradientBrush StartPoint="0,0" EndPoint="0,1">
                    <GradientStop Color="#60FFFFFF" Offset="0.0" />
                    <GradientStop Color="#10FFFFFF" Offset="1.0" />
                </LinearGradientBrush>
            </Border.BorderBrush>
            <Border.BorderThickness>1,1,1,1</Border.BorderThickness>
            
            <!-- Красивая мягкая тень -->
            <Border.Effect>
                <DropShadowEffect Color="Black" BlurRadius="25" ShadowDepth="10" Opacity="0.3" Direction="270"/>
            </Border.Effect>

            <!-- Контейнер для будущих иконок -->
            <StackPanel x:Name="DockIconsPanel" Orientation="Horizontal" HorizontalAlignment="Center" VerticalAlignment="Center">
                <TextBlock Text="Подготовка системы завершена!" 
                           Foreground="#E0FFFFFF" 
                           FontFamily="Segoe UI" 
                           FontSize="16" 
                           FontWeight="SemiBold"
                           Margin="20,0,20,0">
                    <TextBlock.Effect>
                        <DropShadowEffect Color="Black" BlurRadius="5" ShadowDepth="1" Opacity="0.5"/>
                    </TextBlock.Effect>
                </TextBlock>
            </StackPanel>
        </Border>
    </Grid>
</Window>
