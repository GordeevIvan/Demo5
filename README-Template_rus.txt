
# Название проекта *ООО_*
## Запуск приложения 
Для запуска приложения переходим в каталог *Название проекта*\bin\Debug и запускаем файл *название проекта*.ехе
## Средства разработки 
- MySql Workbench
- Visual Studio 2022
- C#
## Автор
*Гордеев Иван* 20ИС-1
















































































create database Trade000;
use Trade000;
create table Roles
(
RoleID int primary key auto_increment,
RoleName nvarchar(100) not null
);
create table Users
(
UserID int primary key auto_increment,
UserSurname nvarchar(100) not null,
UserName nvarchar(100) not null,
UserPatronymic nvarchar(100) not null,
UserLogin nvarchar(100) not null,
UserPassword nvarchar(100) not null,
UserRole int not null,
foreign key (UserRole) references Roles(RoleID)
);

create table Units
(
UnitID int primary key auto_increment,
ProductUnit nvarchar(100) not null
);

create table Categorys
(
CategoryID int primary key auto_increment,
ProductCategory nvarchar(100) not null
);

create table Suppliers
(
SupplierID int primary key auto_increment,
ProductSupplier nvarchar(100) not null
);

create table Manufacturers
(
ManufacturerID int primary key auto_increment,
ProductManufacturer nvarchar(100) not null
);

create table Products
(
ProductArticleNumber nvarchar(100) primary key,
ProductName nvarchar(100) not null,
ProductDescription nvarchar(500) not null,
ProductCategory int not null,
ProductPhoto longblob,
ProductManufacturer int not null,
ProductCost decimal(19,4) not null,
ProductDiscountAmount tinyint null,
ProductQuantityInStock int not null,
ProductUnit int not null,
ProductSupplier int not null,
ProductMaxDiscountAmount tinyint null,
foreign key (ProductCategory) references Categorys(CategoryID),
foreign key (ProductManufacturer) references Manufacturers(ManufacturerID),
foreign key (ProductUnit) references Units(UnitID),
foreign key (ProductSupplier) references Suppliers(SupplierID)
);



-------------------------------------------------------------------------------------------------------------------------------------------------------------
Scaffold-DbContext "Server=172.17.4.11;Port=3306;User=user_20is_6;Password=2143;Database=user_20is_6" "Pomelo.EntityFrameworkCore.MySql" -OutputDir "Model"
----------------------------------------------------------------------------------------------------------
В APP.xaml
---------------------------------------------------------------------------------
public static trade000Context context { get; } = new trade000Context();


Класс ConstantData
================================================================================================================================================================================
 class ConstantData
    {
        public static string Atricle;
        public static int All;
    }


Окно Product
XAML
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 Title="Продукты" MinHeight="800" MinWidth="1400" FontSize="20" FontFamily="Comic Sans MS" WindowStartupLocation="CenterScreen">
    <Grid>
        <StackPanel HorizontalAlignment="Center" VerticalAlignment="Center" Height="784">
            <Label x:Name="name" Content="" HorizontalAlignment="Right"/>
            <StackPanel  Background="#FFFFCC99" Orientation="Horizontal" HorizontalAlignment="Center">
                <TextBlock VerticalAlignment="Center">Поиск:</TextBlock>
                <Label/>
                <TextBox x:Name="Search" Width="300" TextChanged="Search_TextChanged"/>
                <Label/>
                <TextBlock VerticalAlignment="Center">Фильтрация:</TextBlock>
                <Label/>
                <ComboBox x:Name="Filtration" Width="200" DropDownClosed="Filtration_DropDownClosed">
                    <ComboBoxItem>Все диапазоны</ComboBoxItem>
                    <ComboBoxItem>0-9.99%</ComboBoxItem>
                    <ComboBoxItem>10-14.99%</ComboBoxItem>
                    <ComboBoxItem>15% и больше</ComboBoxItem>
                </ComboBox>
                <Label/>
                <TextBlock VerticalAlignment="Center">Сортировка:</TextBlock>
                <Label/>
                <StackPanel>
                    <RadioButton x:Name="Asc" Checked="Asc_Checked">По возрастанию</RadioButton>
                    <RadioButton x:Name="Desc" Checked="Desc_Checked">По убыванию</RadioButton>
                </StackPanel>
            </StackPanel>
            <Label/>
            <TextBlock  Foreground="Black" x:Name="str"/>
            <ScrollViewer Height="614" Width="1400">
                <StackPanel x:Name="ProductView"/>
            </ScrollViewer>
                    <Button x:Name="exit" Background="#FFCC6600" FontFamily="Comic Sans MS" Click="exit_Click">Выход</Button>
          </StackPanel>
    </Grid>
</Window>

Код: 
--------------------------------------------------------------------------------------------------------------------------------------------------
 InitializeComponent();
            List<Models.Products> list = App.context.Products.ToList();
            Classes.ConstantData.All = list.Count;
            LoadData();
        }

        private void Desc_Checked(object sender, RoutedEventArgs e)
        {
            LoadData();
        }

        private void Asc_Checked(object sender, RoutedEventArgs e)
        {
            LoadData();
        }

        private void Filtration_DropDownClosed(object sender, EventArgs e)
        {
            LoadData();
        }

        private void Search_TextChanged(object sender, TextChangedEventArgs e)
        {
            LoadData();
        }
        public void LoadData()
        {
            ProductView.Children.Clear();
            List<Models.Products> list = App.context.Products.ToList();
            if (Filtration.SelectedItem != null)
            {
                switch (Filtration.SelectedIndex)
                {
                    case 0:
                        list = App.context.Products.ToList();
                        break;
                    case 1:
                        list = list.Where(p => p.ProductDiscountAmount > 0 && p.ProductDiscountAmount < Convert.ToDecimal(9.99)).ToList();
                        break;
                    case 2:
                        list = list.Where(p => p.ProductDiscountAmount > 10 && p.ProductDiscountAmount < Convert.ToDecimal(14.99)).ToList();
                        break;
                    case 3:
                        list = list.Where(p => p.ProductDiscountAmount >= 15 && p.ProductDiscountAmount < 100).ToList();
                        break;
                }
            }
            if (Search.Text.Length != 0)
                list = list.Where(p => p.ProductName.Contains(Search.Text)).ToList();
            if (Asc.IsChecked == true)
                list = list.OrderBy(p => p.ProductCost).ToList();
            else if (Desc.IsChecked == true)
                list = list.OrderByDescending(p => p.ProductCost).ToList();

            foreach (var item in list)
                ProductView.Children.Add(new UserControls.ProductCard(item.ProductArticleNumber, item.ProductName, item.ProductDescription, item.ProductManufacturer.ToString(), item.ProductCost.ToString(), Convert.ToDecimal(item.ProductDiscountAmount), item.ProductPhoto));
            str.Text = list.Count.ToString() + "из" + Classes.ConstantData.All.ToString();
        }


        private void exit_Click(object sender, RoutedEventArgs e)
        {
            this.Close();
        }
    }

}



UserControls

ProductCard.xaml
====================================================================================================================================
 d:DesignHeight="250" d:DesignWidth="1300" FontSize="20" FontFamily="Comic Sans MS">
    <Grid Height="250" Width="1300">
        <Grid Background="Black" Height="240" Width="1290" MouseDown="Grid_MouseDown">
            <Grid Background="White" Height="230" Width="1280">
                <Label x:Name="SaleColor" Height="0" Background=" #7fff00"/>
                <StackPanel HorizontalAlignment="Center" VerticalAlignment="Center" Orientation="Horizontal">
                    <Grid Width="320" Height="220" Background="Black">
                        <Grid Width="310" Height="210" Background="White">
                            <Image x:Name="Imgs" Width="400" Height="200" Source="/Images/picture.png"/>
                        </Grid>
                    </Grid>
                    <Label/>
                    <Grid Height="220" Width="700" Background="Black">
                        <Grid Height="210" Width="690" Background="White">
                            <StackPanel Width="680" Height="200">
                                <TextBlock x:Name="Title" FontWeight="Bold">Название</TextBlock>
                                <Label/>
                                <TextBlock x:Name="Description" Width="680" TextWrapping="Wrap" Height="90">Описание</TextBlock>
                                <Label/>
                                <TextBlock x:Name="Manufacturer">Производитель:</TextBlock>
                                <Label/>
                                <StackPanel Orientation="Horizontal">
                                    <TextBlock>Цена:</TextBlock>
                                    <TextBlock x:Name="Cost"/>
                                    <Label/>
                                    <TextBlock x:Name="CostDiscount" Visibility="Hidden"/>
                                </StackPanel>
                            </StackPanel>
                        </Grid>
                    </Grid>
                    <Label/>
                    <Grid Height="150" Width="200" Background="Black">
                        <Grid Height="140" Width="190" Background="White" x:Name="BackgroundSale">
                            <Label FontSize="19">Размер скидки</Label>
                            <TextBlock x:Name="Sale1" VerticalAlignment="Center" HorizontalAlignment="Right" TextWrapping="Wrap" Width="180"></TextBlock>
                        </Grid>
                    </Grid>
                </StackPanel>
                <TextBlock x:Name="Strike" TextDecorations="Strikethrough" Width="0"/>
            </Grid>
        </Grid>
    </Grid>
</UserControl>

КОД
----------------------------------------------------------------------------------------------------------------------------------------------------
  public partial class ProductCard : UserControl
    {
        string Article;
        public ProductCard(string Atricle, string Title, string Description, string Manufacture, string Cost, decimal Sale, byte[] Img)
        {
            InitializeComponent();
            this.Article = Atricle;
            this.Title.Text = Title;
            this.Description.Text = Description;
            Manufacturers manufacturers = App.context.Manufacturers.ToList().Find(m => m.ManufacturerId.ToString() == Manufacture);
            this.Manufacturer.Text += manufacturers.ProductManufacturer;
            this.Cost.Text += Cost.Split(',')[0] + " р.";
            this.CostDiscount.Text += (Convert.ToDecimal(Cost) - Convert.ToDecimal(Cost) * Sale / 100).ToString().Split(',')[0] + " р.";
            this.Cost.TextDecorations = Strike.TextDecorations;
            if (Sale > 15)
                BackgroundSale.Background = SaleColor.Background;
            if (Sale > 0)
            {
                CostDiscount.Visibility = Visibility.Visible;
                Sale1.Text += Sale + "%";
            }

            try
            {
                BitmapImage img = new BitmapImage();
                MemoryStream MS = new MemoryStream(Img);
                img.BeginInit();
                img.StreamSource = MS;
                img.EndInit();
                this.Imgs.Source = img;
            }
            catch
            {
            }
        }

        private void Grid_MouseDown(object sender, MouseButtonEventArgs e)
        {
            Classes.ConstantData.Atricle = Article;
            MessageBox.Show("Продукт с артикулом " + Article + " выбран для работы!");
        }
    }
}