<Window x:Class="MacDock.SettingsWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="Настройки MacDock" Height="380" Width="350"
        WindowStartupLocation="CenterScreen" WindowStyle="ToolWindow"
        Background="#1A1A1A" Foreground="White">
    <Grid Margin="20">
        <StackPanel>
            <TextBlock Text="⚙ Кастомизация Дока" FontSize="22" FontWeight="Bold" Margin="0,0,0,25" HorizontalAlignment="Center"/>
            
            <TextBlock Text="Размер иконок:" FontSize="14" Foreground="#AAAAAA" Margin="0,0,0,5"/>
            <Slider x:Name="SizeSlider" Minimum="30" Maximum="80" Value="40" TickFrequency="1" IsSnapToTickEnabled="True" Margin="0,0,0,20"/>

            <TextBlock Text="Прозрачность панели:" FontSize="14" Foreground="#AAAAAA" Margin="0,0,0,5"/>
            <Slider x:Name="OpacitySlider" Minimum="0.1" Maximum="1.0" Value="0.85" TickFrequency="0.05" IsSnapToTickEnabled="True" Margin="0,0,0,20"/>

            <TextBlock Text="Высота над панелью задач:" FontSize="14" Foreground="#AAAAAA" Margin="0,0,0,5"/>
            <Slider x:Name="MarginSlider" Minimum="0" Maximum="100" Value="10" TickFrequency="1" IsSnapToTickEnabled="True" Margin="0,0,0,30"/>

            <Button Content="Сохранить и Применить" Height="40" Background="#0078D7" Foreground="White" FontSize="16" FontWeight="SemiBold" BorderThickness="0" Cursor="Hand" Click="Save_Click">
                <Button.Resources>
                    <Style TargetType="Border">
                        <Setter Property="CornerRadius" Value="8"/>
                    </Style>
                </Button.Resources>
            </Button>
        </StackPanel>
    </Grid>
</Window>
