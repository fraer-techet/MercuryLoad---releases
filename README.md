<Window x:Class="MacDock.SettingsWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="Настройки MacDock" Height="450" Width="400"
        WindowStartupLocation="CenterScreen"
        Background="#1A1A1A" Foreground="White">
    <Grid Margin="20">
        <StackPanel>
            <TextBlock Text="Настройки Дока" FontSize="24" FontWeight="Bold" Margin="0,0,0,20"/>
            
            <TextBlock Text="Твои приложения:" FontSize="14" Foreground="#AAAAAA" Margin="0,0,0,10"/>
            
            <!-- Список твоих программ -->
            <ListBox x:Name="AppsListBox" Height="200" Background="#2A2A2A" Foreground="White" BorderThickness="0" Margin="0,0,0,15">
                <ListBox.ItemTemplate>
                    <DataTemplate>
                        <TextBlock Text="{Binding}" Padding="5"/>
                    </DataTemplate>
                </ListBox.ItemTemplate>
            </ListBox>

            <!-- Кнопка добавления -->
            <Button Content="+ Добавить .exe приложение" Height="35" Background="#0078D7" Foreground="White" BorderThickness="0" Cursor="Hand" Click="AddApp_Click"/>
            
            <!-- Кнопка сохранения -->
            <Button Content="Сохранить и перезапустить Док" Height="35" Background="#28A745" Foreground="White" BorderThickness="0" Margin="0,15,0,0" Cursor="Hand" Click="Save_Click"/>
        </StackPanel>
    </Grid>
</Window>
