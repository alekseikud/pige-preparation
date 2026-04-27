Below is the **full WPF lab guideline from zero**, stage by stage. Each stage has:

```text
What to create
What code to write
What result should look like
Why it works
```

Based on your PDF task. 

---

# Stage 0 — Project structure

Create WPF project:

```text
ContactManager
├── MainWindow.xaml
├── MainWindow.xaml.cs
├── Contact.cs
├── AddContactWindow.xaml
├── AddContactWindow.xaml.cs
└── Resources
    ├── man.png
    └── woman.jpg
```

The final app should work like this:

```text
MainWindow
├── Menu
│   ├── File
│   ├── Contacts
│   └── About
│
├── List tab
│   └── visual contact list
│
└── Grid tab
    └── editable DataGrid
```

---

# Stage 1 — `Contact.cs`

Create file:

```text
Contact.cs
```

Write:

```csharp
namespace ContactManager;

public class Contact
{
    public string? Name { get; set; }
    public string? Surname { get; set; }
    public string? Email { get; set; }
    public string? Phone { get; set; }
    public Gender Gender { get; set; }
}

public enum Gender
{
    Male,
    Female
}
```

## Result

Nothing visual yet.

Internally you now have:

```text
Contact
├── Name
├── Surname
├── Email
├── Phone
└── Gender
```

---

# Stage 2 — Basic `MainWindow.xaml`

Write this first:

```xml
<Window x:Class="ContactManager.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="Contact Manager"
        Width="800"
        Height="600"
        MinWidth="500"
        MinHeight="500"
        WindowStartupLocation="CenterScreen">

    <Grid>
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="*"/>
        </Grid.RowDefinitions>

        <Menu Grid.Row="0" Background="LightGray">
            <MenuItem Header="File">
                <MenuItem Header="Import" IsEnabled="False"/>
                <MenuItem Header="Export" IsEnabled="False"/>
                <Separator/>
                <MenuItem Header="Exit" Click="MenuItem_Exit"/>
            </MenuItem>

            <MenuItem Header="Contacts">
                <MenuItem Header="Add contact" Click="MenuItem_AddContact"/>
                <MenuItem Header="Clear contacts" Click="MenuItem_ClearContacts"/>
            </MenuItem>

            <MenuItem Header="About" Click="MenuItem_About"/>
        </Menu>

        <TabControl Grid.Row="1">
            <TabItem Header="List">
                <ListBox/>
            </TabItem>

            <TabItem Header="Grid">
                <DataGrid/>
            </TabItem>
        </TabControl>
    </Grid>
</Window>
```

## Result picture

```text
+------------------------------------------------+
| File | Contacts | About                        |
+------------------------------------------------+
| [List] [Grid]                                  |
|                                                |
| empty area                                     |
|                                                |
+------------------------------------------------+
```

## Important

Do **not** add this:

```xml
d:HorizontalAlignment="Center"
d:VerticalAlignment="Center"
```

It caused your menu problem.

---

# Stage 3 — Basic `MainWindow.xaml.cs`

Write:

```csharp
using System.Collections.ObjectModel;
using System.Windows;

namespace ContactManager;

public partial class MainWindow : Window
{
    public ObservableCollection<Contact> Contacts { get; set; }

    public MainWindow()
    {
        InitializeComponent();

        Contacts = new ObservableCollection<Contact>
        {
            new Contact
            {
                Name = "John",
                Surname = "Smith",
                Email = "john@mail.com",
                Phone = "123-456-789",
                Gender = Gender.Male
            },
            new Contact
            {
                Name = "Anna",
                Surname = "Brown",
                Email = "anna@mail.com",
                Phone = "987-654-321",
                Gender = Gender.Female
            }
        };

        DataContext = Contacts;
    }

    private void MenuItem_Exit(object sender, RoutedEventArgs e)
    {
        Close();
    }

    private void MenuItem_AddContact(object sender, RoutedEventArgs e)
    {
        MessageBox.Show("Add contact clicked");
    }

    private void MenuItem_ClearContacts(object sender, RoutedEventArgs e)
    {
        Contacts.Clear();
    }

    private void MenuItem_About(object sender, RoutedEventArgs e)
    {
        MessageBox.Show(
            "This is simple contact manager.",
            "Contact Manager",
            MessageBoxButton.OK,
            MessageBoxImage.Information
        );
    }
}
```

## Why `ObservableCollection`

This is the core of the whole lab:

```text
Contacts collection changes
        ↓
WPF notices automatically
        ↓
ListBox / DataGrid update
```

If you use `List<Contact>`, changes will not update the UI properly.

---

# Stage 4 — Bind the DataGrid

In `MainWindow.xaml`, replace the empty DataGrid:

```xml
<DataGrid/>
```

with:

```xml
<DataGrid ItemsSource="{Binding}"
          AutoGenerateColumns="True"/>
```

Full Grid tab:

```xml
<TabItem Header="Grid">
    <DataGrid ItemsSource="{Binding}"
              AutoGenerateColumns="True"/>
</TabItem>
```

## Result picture

```text
Grid tab:

+------+---------+---------------+-------------+--------+
| Name | Surname | Email         | Phone       | Gender |
+------+---------+---------------+-------------+--------+
| John | Smith   | john@mail.com | 123-456-789 | Male   |
| Anna | Brown   | anna@mail.com | 987-654-321 | Female |
+------+---------+---------------+-------------+--------+
```

## Why it works

This:

```xml
ItemsSource="{Binding}"
```

means:

```text
Use current DataContext
```

And your `DataContext` is:

```csharp
Contacts
```

So:

```text
DataGrid displays Contacts
```

---

# Stage 5 — Bind ListBox and show selected contact

Replace the List tab with:

```xml
<TabItem Header="List">
    <Grid>
        <Grid.RowDefinitions>
            <RowDefinition Height="*"/>
            <RowDefinition Height="Auto"/>
        </Grid.RowDefinitions>

        <ListBox x:Name="ContactsListBox"
                 Grid.Row="0"
                 ItemsSource="{Binding}"
                 DisplayMemberPath="Name"/>

        <StackPanel Grid.Row="1" Margin="10">
            <TextBlock FontWeight="Bold" Text="Selected contact:"/>

            <TextBlock Text="{Binding SelectedItem.Name, ElementName=ContactsListBox}"/>
            <TextBlock Text="{Binding SelectedItem.Surname, ElementName=ContactsListBox}"/>
            <TextBlock Text="{Binding SelectedItem.Email, ElementName=ContactsListBox}"/>
            <TextBlock Text="{Binding SelectedItem.Phone, ElementName=ContactsListBox}"/>
            <TextBlock Text="{Binding SelectedItem.Gender, ElementName=ContactsListBox}"/>
        </StackPanel>
    </Grid>
</TabItem>
```

## Result picture

```text
List tab:

+----------------------+
| John                 |
| Anna                 |
+----------------------+

Selected contact:
John
Smith
john@mail.com
123-456-789
Male
```

## Why it works

This:

```xml
SelectedItem.Name
```

means:

```text
Take selected contact from ListBox
then read its Name
```

Flow:

```text
click John
   ↓
ContactsListBox.SelectedItem = John
   ↓
TextBlocks update
```

---

# Stage 6 — Add real visual ListBox template

Now improve the ListBox so it shows:

```text
Name Surname + avatar
```

First, add images:

```text
Resources/man.png
Resources/woman.jpg
```

In Visual Studio:

```text
Right click image → Properties → Build Action → Resource
```

Now add this inside `<Window>` before `<Grid>`:

```xml
<Window.Resources>
    <BitmapImage x:Key="WomanAvatar" UriSource="/Resources/woman.jpg"/>
    <BitmapImage x:Key="ManAvatar" UriSource="/Resources/man.png"/>

    <DataTemplate x:Key="ContactListItemTemplate">
        <Grid HorizontalAlignment="Stretch">
            <Grid.ColumnDefinitions>
                <ColumnDefinition Width="*"/>
                <ColumnDefinition Width="Auto"/>
            </Grid.ColumnDefinitions>

            <StackPanel Grid.Column="0"
                        Orientation="Horizontal"
                        VerticalAlignment="Center">
                <Label Content="{Binding Name}" FontSize="20"/>
                <Label Content="{Binding Surname}" FontSize="20"/>
            </StackPanel>

            <Border Grid.Column="1"
                    Width="50"
                    Height="50"
                    BorderBrush="Black"
                    BorderThickness="1"
                    Margin="3">
                <Image Stretch="Fill">
                    <Image.Style>
                        <Style TargetType="Image">
                            <Style.Triggers>
                                <DataTrigger Binding="{Binding Gender}" Value="Female">
                                    <Setter Property="Source" Value="{StaticResource WomanAvatar}"/>
                                </DataTrigger>

                                <DataTrigger Binding="{Binding Gender}" Value="Male">
                                    <Setter Property="Source" Value="{StaticResource ManAvatar}"/>
                                </DataTrigger>
                            </Style.Triggers>
                        </Style>
                    </Image.Style>
                </Image>
            </Border>
        </Grid>
    </DataTemplate>

    <Style x:Key="ContactListItemStyle" TargetType="ListBoxItem">
        <Style.Triggers>
            <Trigger Property="ItemsControl.AlternationIndex" Value="0">
                <Setter Property="Background" Value="#FFAFC5FF"/>
            </Trigger>

            <Trigger Property="ItemsControl.AlternationIndex" Value="1">
                <Setter Property="Background" Value="#FF75A1FF"/>
            </Trigger>
        </Style.Triggers>
    </Style>
</Window.Resources>
```

Now replace the ListBox with:

```xml
<ListBox x:Name="ContactsListBox"
         Grid.Row="0"
         ItemsSource="{Binding}"
         ItemTemplate="{StaticResource ContactListItemTemplate}"
         ItemContainerStyle="{StaticResource ContactListItemStyle}"
         AlternationCount="2"
         HorizontalContentAlignment="Stretch"/>
```

## Result picture

```text
List tab:

+---------------------------------------------+
| John Smith                         [man]    | blue
+---------------------------------------------+
| Anna Brown                         [woman]  | darker blue
+---------------------------------------------+

Selected contact:
John
Smith
john@mail.com
123-456-789
Male
```

## Why `AlternationCount`

This makes WPF assign item indexes:

```text
0, 1, 0, 1, 0, 1...
```

Then your style colors them differently.

Do **not** color manually in C#.

Bad:

```csharp
if (Contacts.Count % 2 == 0)
```

That breaks after delete/sort.

---

# Stage 7 — Create `AddContactWindow.xaml`

Create new WPF Window:

```text
AddContactWindow.xaml
```

Write:

```xml
<Window x:Class="ContactManager.AddContactWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:local="clr-namespace:ContactManager"
        xmlns:System="clr-namespace:System;assembly=mscorlib"
        Title="Add contact"
        Height="250"
        Width="350"
        WindowStartupLocation="CenterScreen"
        ResizeMode="NoResize"
        ShowInTaskbar="False">

    <Window.Resources>
        <ObjectDataProvider x:Key="GenderList"
                            MethodName="GetValues"
                            ObjectType="{x:Type System:Enum}">
            <ObjectDataProvider.MethodParameters>
                <x:Type TypeName="local:Gender"/>
            </ObjectDataProvider.MethodParameters>
        </ObjectDataProvider>
    </Window.Resources>

    <Grid>
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="*"/>
            <RowDefinition Height="Auto"/>
        </Grid.RowDefinitions>

        <Label Grid.Row="0"
               HorizontalAlignment="Center"
               FontSize="24">
            Add new contact
        </Label>

        <Grid Grid.Row="1" Margin="10">
            <Grid.ColumnDefinitions>
                <ColumnDefinition Width="Auto"/>
                <ColumnDefinition Width="*"/>
            </Grid.ColumnDefinitions>

            <Grid.RowDefinitions>
                <RowDefinition/>
                <RowDefinition/>
                <RowDefinition/>
                <RowDefinition/>
                <RowDefinition/>
            </Grid.RowDefinitions>

            <Label Grid.Column="0" Grid.Row="0">Name:</Label>
            <Label Grid.Column="0" Grid.Row="1">Surname:</Label>
            <Label Grid.Column="0" Grid.Row="2">Email:</Label>
            <Label Grid.Column="0" Grid.Row="3">Phone:</Label>
            <Label Grid.Column="0" Grid.Row="4">Gender:</Label>

            <TextBox Grid.Column="1" Grid.Row="0"
                     Text="{Binding Name, UpdateSourceTrigger=PropertyChanged}"/>

            <TextBox Grid.Column="1" Grid.Row="1"
                     Text="{Binding Surname, UpdateSourceTrigger=PropertyChanged}"/>

            <TextBox Grid.Column="1" Grid.Row="2"
                     Text="{Binding Email, UpdateSourceTrigger=PropertyChanged}"/>

            <TextBox Grid.Column="1" Grid.Row="3"
                     Text="{Binding Phone, UpdateSourceTrigger=PropertyChanged}"/>

            <ComboBox Grid.Column="1" Grid.Row="4"
                      ItemsSource="{Binding Source={StaticResource GenderList}}"
                      SelectedItem="{Binding Gender}"/>
        </Grid>

        <StackPanel Grid.Row="2"
                    Orientation="Horizontal"
                    HorizontalAlignment="Center"
                    Margin="0,10,0,10">

            <Button Width="100"
                    Height="30"
                    Click="AddContact">
                Add contact
            </Button>

            <Button Width="100"
                    Height="30"
                    Margin="15,0,0,0"
                    Click="Cancel">
                Cancel
            </Button>
        </StackPanel>
    </Grid>
</Window>
```

## Result picture

```text
+--------------------------------+
|        Add new contact         |
|                                |
| Name:    [______________]      |
| Surname: [______________]      |
| Email:   [______________]      |
| Phone:   [______________]      |
| Gender:  [Male/Female v]      |
|                                |
| [Add contact] [Cancel]         |
+--------------------------------+
```

---

# Stage 8 — `AddContactWindow.xaml.cs`

Write:

```csharp
using System.Windows;

namespace ContactManager;

public partial class AddContactWindow : Window
{
    public Contact NewContact { get; private set; }

    public AddContactWindow()
    {
        InitializeComponent();

        NewContact = new Contact();
        DataContext = NewContact;
    }

    private void AddContact(object sender, RoutedEventArgs e)
    {
        DialogResult = true;
        Close();
    }

    private void Cancel(object sender, RoutedEventArgs e)
    {
        DialogResult = false;
        Close();
    }
}
```

## Why it works

This:

```csharp
DataContext = NewContact;
```

connects all fields to one object.

So this:

```xml
Text="{Binding Name}"
```

means:

```text
TextBox writes into NewContact.Name
```

The button does not manually read fields.

Correct flow:

```text
User types
   ↓
Binding updates NewContact
   ↓
User clicks Add
   ↓
DialogResult = true
   ↓
MainWindow receives NewContact
```

---

# Stage 9 — Connect Add Contact menu

In `MainWindow.xaml.cs`, replace this:

```csharp
private void MenuItem_AddContact(object sender, RoutedEventArgs e)
{
    MessageBox.Show("Add contact clicked");
}
```

with:

```csharp
private void MenuItem_AddContact(object sender, RoutedEventArgs e)
{
    Opacity = 0.5;

    var addContactWindow = new AddContactWindow();

    bool? result = addContactWindow.ShowDialog();

    if (result == true)
    {
        Contacts.Add(addContactWindow.NewContact);
    }

    Opacity = 1;
}
```

## Result picture

```text
Contacts → Add contact
        ↓
+---------------------------+
| Add new contact window    |
+---------------------------+
        ↓
User clicks Add
        ↓
Contact appears in List tab and Grid tab
```

## Why it works

`ShowDialog()` blocks the main window.

```text
AddContactWindow
        ↓ returns Contact
MainWindow
        ↓ adds to ObservableCollection
ListBox + DataGrid update automatically
```

---

# Stage 10 — Final `MainWindow.xaml`

Full file:

```xml
<Window x:Class="ContactManager.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="Contact Manager"
        Width="800"
        Height="600"
        MinWidth="500"
        MinHeight="500"
        WindowStartupLocation="CenterScreen">

    <Window.Resources>
        <BitmapImage x:Key="WomanAvatar" UriSource="/Resources/woman.jpg"/>
        <BitmapImage x:Key="ManAvatar" UriSource="/Resources/man.png"/>

        <DataTemplate x:Key="ContactListItemTemplate">
            <Grid HorizontalAlignment="Stretch">
                <Grid.ColumnDefinitions>
                    <ColumnDefinition Width="*"/>
                    <ColumnDefinition Width="Auto"/>
                </Grid.ColumnDefinitions>

                <StackPanel Grid.Column="0"
                            Orientation="Horizontal"
                            VerticalAlignment="Center">
                    <Label Content="{Binding Name}" FontSize="20"/>
                    <Label Content="{Binding Surname}" FontSize="20"/>
                </StackPanel>

                <Border Grid.Column="1"
                        Width="50"
                        Height="50"
                        BorderBrush="Black"
                        BorderThickness="1"
                        Margin="3">
                    <Image Stretch="Fill">
                        <Image.Style>
                            <Style TargetType="Image">
                                <Style.Triggers>
                                    <DataTrigger Binding="{Binding Gender}" Value="Female">
                                        <Setter Property="Source" Value="{StaticResource WomanAvatar}"/>
                                    </DataTrigger>

                                    <DataTrigger Binding="{Binding Gender}" Value="Male">
                                        <Setter Property="Source" Value="{StaticResource ManAvatar}"/>
                                    </DataTrigger>
                                </Style.Triggers>
                            </Style>
                        </Image.Style>
                    </Image>
                </Border>
            </Grid>
        </DataTemplate>

        <Style x:Key="ContactListItemStyle" TargetType="ListBoxItem">
            <Style.Triggers>
                <Trigger Property="ItemsControl.AlternationIndex" Value="0">
                    <Setter Property="Background" Value="#FFAFC5FF"/>
                </Trigger>

                <Trigger Property="ItemsControl.AlternationIndex" Value="1">
                    <Setter Property="Background" Value="#FF75A1FF"/>
                </Trigger>
            </Style.Triggers>
        </Style>
    </Window.Resources>

    <Grid>
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="*"/>
        </Grid.RowDefinitions>

        <Menu Grid.Row="0" Background="LightGray">
            <MenuItem Header="File">
                <MenuItem Header="Import" IsEnabled="False"/>
                <MenuItem Header="Export" IsEnabled="False"/>
                <Separator/>
                <MenuItem Header="Exit" Click="MenuItem_Exit"/>
            </MenuItem>

            <MenuItem Header="Contacts">
                <MenuItem Header="Add contact" Click="MenuItem_AddContact"/>
                <MenuItem Header="Clear contacts" Click="MenuItem_ClearContacts"/>
            </MenuItem>

            <MenuItem Header="About" Click="MenuItem_About"/>
        </Menu>

        <TabControl Grid.Row="1">
            <TabItem Header="List">
                <Grid>
                    <Grid.RowDefinitions>
                        <RowDefinition Height="*"/>
                        <RowDefinition Height="Auto"/>
                    </Grid.RowDefinitions>

                    <ListBox x:Name="ContactsListBox"
                             Grid.Row="0"
                             ItemsSource="{Binding}"
                             ItemTemplate="{StaticResource ContactListItemTemplate}"
                             ItemContainerStyle="{StaticResource ContactListItemStyle}"
                             AlternationCount="2"
                             HorizontalContentAlignment="Stretch"/>

                    <StackPanel Grid.Row="1" Margin="10">
                        <TextBlock FontWeight="Bold" Text="Selected contact:"/>

                        <TextBlock Text="{Binding SelectedItem.Name, ElementName=ContactsListBox}"/>
                        <TextBlock Text="{Binding SelectedItem.Surname, ElementName=ContactsListBox}"/>
                        <TextBlock Text="{Binding SelectedItem.Email, ElementName=ContactsListBox}"/>
                        <TextBlock Text="{Binding SelectedItem.Phone, ElementName=ContactsListBox}"/>
                        <TextBlock Text="{Binding SelectedItem.Gender, ElementName=ContactsListBox}"/>
                    </StackPanel>
                </Grid>
            </TabItem>

            <TabItem Header="Grid">
                <DataGrid ItemsSource="{Binding}"
                          AutoGenerateColumns="True"/>
            </TabItem>
        </TabControl>
    </Grid>
</Window>
```

---

# Stage 11 — Final `MainWindow.xaml.cs`

Full file:

```csharp
using System.Collections.ObjectModel;
using System.Windows;

namespace ContactManager;

public partial class MainWindow : Window
{
    public ObservableCollection<Contact> Contacts { get; set; }

    public MainWindow()
    {
        InitializeComponent();

        Contacts = new ObservableCollection<Contact>
        {
            new Contact
            {
                Name = "John",
                Surname = "Smith",
                Email = "john@mail.com",
                Phone = "123-456-789",
                Gender = Gender.Male
            },
            new Contact
            {
                Name = "Anna",
                Surname = "Brown",
                Email = "anna@mail.com",
                Phone = "987-654-321",
                Gender = Gender.Female
            }
        };

        DataContext = Contacts;
    }

    private void MenuItem_Exit(object sender, RoutedEventArgs e)
    {
        Close();
    }

    private void MenuItem_AddContact(object sender, RoutedEventArgs e)
    {
        Opacity = 0.5;

        var addContactWindow = new AddContactWindow();

        bool? result = addContactWindow.ShowDialog();

        if (result == true)
        {
            Contacts.Add(addContactWindow.NewContact);
        }

        Opacity = 1;
    }

    private void MenuItem_ClearContacts(object sender, RoutedEventArgs e)
    {
        Contacts.Clear();
    }

    private void MenuItem_About(object sender, RoutedEventArgs e)
    {
        MessageBox.Show(
            "This is simple contact manager.",
            "Contact Manager",
            MessageBoxButton.OK,
            MessageBoxImage.Information
        );
    }
}
```

---

# Final result picture

```text
+-------------------------------------------------------------+
| File | Contacts | About                                     |
+-------------------------------------------------------------+
| [List] [Grid]                                               |
|                                                             |
| +---------------------------------------------------------+ |
| | John Smith                                      [man]    | |
| +---------------------------------------------------------+ |
| | Anna Brown                                      [woman]  | |
| +---------------------------------------------------------+ |
|                                                             |
| Selected contact:                                           |
| John                                                        |
| Smith                                                       |
| john@mail.com                                               |
| 123-456-789                                                 |
| Male                                                        |
+-------------------------------------------------------------+
```

Grid tab:

```text
+------+---------+---------------+-------------+--------+
| Name | Surname | Email         | Phone       | Gender |
+------+---------+---------------+-------------+--------+
| John | Smith   | john@mail.com | 123-456-789 | Male   |
| Anna | Brown   | anna@mail.com | 987-654-321 | Female |
+------+---------+---------------+-------------+--------+
```

---

# Mental model of the whole app

```text
AddContactWindow
      |
      | creates Contact
      v
MainWindow
      |
      | Contacts.Add(...)
      v
ObservableCollection<Contact>
      |
      +------------------+
      |                  |
      v                  v
ListBox              DataGrid
visual view          editing table
```

That is the whole base lab. Everything depends on this one shared collection:

```csharp
ObservableCollection<Contact> Contacts
```
