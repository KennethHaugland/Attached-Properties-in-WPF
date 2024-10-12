# Attached-Properties-in-WPF

<ul class="download">
	<li><a href="992239/WpfSimpleSearchebleTreeView_V2-noexe.zip">Download source code in VB - 183 KB</a></li>
	<li><a href="WpfSimpleSearchebleTreeView_V2.zip">Download source code and demo in VB - 183 KB</a></li>
	<li><a href="WpfAttachedPropertiesCS_V2.zip">Download WpfAttachedPropertiesCS_V2.zip</a></li>
</ul>

<p><img src="ProgramScreenShot.gif" style="width: 400px; height: 266px" /></p>

<h2>Introduction</h2>

<p>In this article I will show you examples of three implementations using <a href="https://msdn.microsoft.com/en-us/library/ms749011%28v=vs.110%29.aspx">attached properties</a>, that quite simply lets you add convenient behavior to an existing control (or class) with one line of XAML code, instead of the alternative, to inherit the control in a custom class to just expand it with a property. The usage of attached properties is quite widespread even if you have never heard of them. The most common example I can think of would be Grid.Row and Grid.Column properties. And they are examples of how attached behaviors should work, a reusable code that takes a minimal of time to set up, and solves a reoccurring problem in quite an elegant fashion.</p>

<h2>Background</h2>

<p>An attached property is quite similar to an Extension of an existing class, exemplified below by the function <code>Reverse</code> that can be called on every <code>String</code> in the application now:</p>

<pre lang="vb.net">
Module Extensions
    &lt;System.Runtime.CompilerServices.Extension&gt;
    Public Function Reverse(ByVal OriginalString As String) As String
        Dim Result As New Text.StringBuilder
        Dim chars As Char() = OriginalString.ToCharArray

        For i As Integer = chars.Count - 1 To 0 Step -1
            Result.Append(chars(i))
        Next

        Return Result.ToString
    End Function
End Module</pre>

<p>While this is valid everywhere, the attached property is far more useful, as one can decide which instance of the class you wish to extend. They are also DependencyProperties, which supports binding to other elements as well.</p>

<p>To create an attached dependency property, you create a new class (called <code>MyNewClass</code> here), and declare it like this:</p>

<pre lang="vb.net">
    Public Shared ReadOnly SearchTextProperty As DependencyProperty =
        DependencyProperty.RegisterAttached(&quot;SearchText&quot;,
                                            GetType(String),
                                            GetType(MyNewClass),
                                            New FrameworkPropertyMetadata(
                                                Nothing,
                                                FrameworkPropertyMetadataOptions.AffectsRender,
                                                New PropertyChangedCallback(AddressOf OnSearchTextChanged)))</pre>

<p>The difference between a normal dependency property and an attached is this call with an attached property:</p>

<pre lang="vb.net">
DependencyProperty.RegisterAttached</pre>

<p>and below is valid for a normal dependency property</p>

<pre lang="vb.net">
DependencyProperty.Register</pre>

<p>The attached property is so useful that an abstraction layer have been created to encapsulate it in Blend, called Behavior, and&nbsp;you could read more about it on&nbsp;<a href="http://briannoyes.net/2012/12/20/attached-behaviors-vs-attached-properties-vs-blend-behaviors/">Brian Noyes</a>&nbsp;blog. To get a glimce of how you can create one yourself, you can check out Jason Kemp&#39;s <a href="http://www.ageektrapped.com/blog/the-missing-net-4-cue-banner-in-wpf-i-mean-watermark-in-wpf/">blog post</a>. His code lets you add elements in a way that is similar to the Behavior class, and a VB.NET version is added to the VS2013 project under the folder called Unused.&nbsp;</p>

<p>To use it within the project you can type in the following:</p>

<pre lang="xml">
     &lt;TreeView&gt;
            &lt;local:Behavior.ContentPresenter&gt;
                &lt;StackPanel HorizontalAlignment=&quot;Center&quot; VerticalAlignment=&quot;Center&quot;&gt;
                    &lt;TextBlock Height=&quot;23&quot;&gt;
                        &lt;Run Text=&quot;Show some text&quot; FontSize=&quot;18&quot;/&gt;
                    &lt;/TextBlock&gt;
                &lt;/StackPanel&gt;
            &lt;/local:Behavior.ContentPresenter&gt;
            ....

        &lt;/TreeView&gt;</pre>

<h2>Binding Enum to ComboBox</h2>

<p>Assuming that you want to bind an <code>Enum </code>to a <code>ComboBox </code>in order to show the current value and allow you to change it with the help of changing the <code>SelectedItem </code>in the <code>ComboBox</code>. This problem seems to be a reoccurring theme with several different solutions posted, even some with attached behavior:</p>

<ul>
	<li><a href="http://www.codeproject.com/Articles/29495/Binding-and-Using-Friendly-Enums-in-WPF">Binding and Using Friendly Enums in WPF</a>&nbsp;</li>
	<li><a href="http://www.codeproject.com/Tips/584206/Enum-to-ComboBox-binding">Enum to ComboBox binding</a>&nbsp;&nbsp;</li>
	<li><a href="http://www.codeproject.com/Articles/317144/MVVM-ComboBox-with-Enums">MVVM ComboBox with Enums</a>&nbsp;</li>
</ul>

<p>I will assume that you want to just show the <code>Enum </code>in a <code>ComboBox </code>with a human readable string, and I will also assume that we all know this trick with adding a readable description to the Enum. I will go through it just for the sake of completeness, and you simply do this:</p>

<pre>
    Public Enum TheComboBoxShow
        &lt;System.ComponentModel.DescriptionAttribute(&quot;The show is on&quot;)&gt;
        [On]
        &lt;System.ComponentModel.DescriptionAttribute(&quot;The show is off&quot;)&gt;
        [Off]
    End Enum</pre>

<p>Now, the Description attribute can be collected from the <code>Enum </code>property by the use of a helper function. This particular one I stole from <a href="http://www.codeproject.com/Tips/101247/Human-readable-strings-for-enum-elements">OriginalGriff</a>, but many other have also created similar functions based on the <a href="https://msdn.microsoft.com/en-us/library/z919e8tw.aspx">MSDN example</a>:</p>

<pre lang="vb.net">
    Public Shared Function GetDescription(value As [Enum]) As String
        &#39; Get information on the enum element
        Dim fi As FieldInfo = value.[GetType]().GetField(value.ToString())
        &#39; Get description for elum element
        Dim attributes As DescriptionAttribute() = DirectCast(fi.GetCustomAttributes(GetType(DescriptionAttribute), False), DescriptionAttribute())
        If attributes.Length &gt; 0 Then
            &#39; DescriptionAttribute exists - return that
            Return attributes(0).Description
        End If
        &#39; No Description set - return enum element name
        Return value.ToString()
    End Function</pre>

<p>The next item &nbsp;on our agenda (as a programmer you gotta love this word play! No? Oki, let&#39;s move along) is to show all the <code>Enum&#39;s </code>in the <code>ComboBox</code>.&nbsp;The main way that it seems to be done, at least by a massive amount of Google searches, is by the usage of <code>ObjectDataProvider</code>:</p>

<pre lang="xml">
&lt;UserControl.Resources&gt;
    &lt;ObjectDataProvider MethodName=&quot;GetValues&quot; 
    ObjectType=&quot;{x:Type sys:Enum}&quot; x:Key=&quot;Options&quot;&gt;
        &lt;ObjectDataProvider.MethodParameters&gt;
            &lt;x:Type TypeName=&quot;local:EnumOptions&quot; /&gt;
        &lt;/ObjectDataProvider.MethodParameters&gt;
    &lt;/ObjectDataProvider&gt;
&lt;/UserControl.Resources&gt;</pre>

<p>And then bind to the ComboBox like so:</p>

<pre lang="xml">
&lt;ComboBox x:Name=&quot;cmbOptions&quot;  
    ItemsSource=&quot;{Binding Source={StaticResource Options}}&quot;
    ....
    ....
&lt;/ComboBox&gt;</pre>

<p>Oh my, that&#39;s a lot of writing to show the <code>Enum</code>. Unless you decided to be a programmer because you absolutely loved typing commands to the computer, this is a bit much. I always wanted it to be along the following path; You declared an <code>Enum </code>property inside your class:</p>

<pre lang="vb.net">
Class TheComboBoxShowCase
    Implements System.ComponentModel.INotifyPropertyChanged

  ...

    Private pTheEnumProperty As TheComboBoxShow = TheComboBoxShow.On
    Public Property TheEnumProperty() As TheComboBoxShow
        Get
            Return pTheEnumProperty
        End Get
        Set(ByVal value As TheComboBoxShow)
            pTheEnumProperty = value
            OnPropertyChanged(&quot;TheEnumProperty&quot;)
        End Set
    End Property

    Public Enum TheComboBoxShow
        &lt;DescriptionAttribute(&quot;The show is on&quot;)&gt;
        [On]
        &lt;DescriptionAttribute(&quot;The Show is off&quot;)&gt;
        [Off]
    End Enum

End Class</pre>

<p>Then in XAML you just connected it to the <code>ComboBox </code>with the following command:</p>

<pre lang="xml">
        &lt;ComboBox Name=&quot;cmbEnum&quot; ItemsSource=&quot;{Binding TheEnumProperty}&quot;  /&gt;</pre>

<p>And then your property in your class would get updated if you changed the value in the <code>ComboBox</code>, and if you changed the value in code behind it would update the selection. Well, it can happen with the help of an attached property:</p>

<pre lang="vb.net">
Imports System.Reflection
Imports System.ComponentModel

Public Class EnumToComboBoxBinding

    Private Shared Combo As ComboBox
    Private Shared ComboNameList As List(Of String)
    Private Shared ComboEnumList As List(Of [Enum])

    Public Shared ReadOnly EnumItemsSourceProperty As DependencyProperty =
        DependencyProperty.RegisterAttached(&quot;EnumItemsSource&quot;,
                                            GetType([Enum]),
                                            GetType(EnumToComboBoxBinding),
                                            New FrameworkPropertyMetadata(Nothing,
                                                                          FrameworkPropertyMetadataOptions.BindsTwoWayByDefault,
                                                                          New PropertyChangedCallback(AddressOf OnEnumItemsSourceChanged)))

    ...

    Public Shared Sub OnEnumItemsSourceChanged(sender As DependencyObject, e As DependencyPropertyChangedEventArgs)
        &#39;Store the set enum locally
        Dim TempEnum As [Enum] = sender.GetValue(EnumItemsSourceProperty)

        &#39; First time run trough or the binding source has changed (last one not very likely or impossible?)
        If ComboEnumList Is Nothing OrElse Not ComboEnumList.Contains(TempEnum) Then

            &#39; Remove any previously handlers
            If Combo IsNot Nothing Then
                RemoveHandler Combo.SelectionChanged, AddressOf EnumValueChanged
            End If

            Combo = DirectCast(sender, ComboBox)

            &#39;Clear the lists
            ComboNameList = New List(Of String)
            ComboEnumList = New List(Of [Enum])

            &#39;Get all possible values for the enum type
            Dim Values = [Enum].GetValues(TempEnum.GetType)

            &#39; Loop trough them and store the description 
            &#39; and the Enum type in two separate lists
            For Each Value In Values
                ComboNameList.Add(GetDescription(Value))
                ComboEnumList.Add(Value)
            Next

            &#39;Add a handler if you change the selected value of the ComboBox
            AddHandler Combo.SelectionChanged, AddressOf EnumValueChanged

            &#39; Set the ComboBox&#39;s ItemsSource to the DescriptionAttribute
            Combo.ItemsSource = ComboNameList
        End If

        &#39; Sync the selected value with the Property 
        Combo.SelectedIndex = ComboEnumList.IndexOf(TempEnum)
    End Sub

    Private Shared Sub EnumValueChanged(sender As Object, e As EventArgs)
        &#39; Selected item in the ComboBox has changes, so updates the  Enum DependencyProperty
        SetEnumItemsSource(Combo, ComboEnumList(Combo.SelectedIndex))
    End Sub

    ...

End Class</pre>

<p>The class is very straight forward, and I only eliminated the functions &quot;we all know by now&quot;. All the <code>Enum </code>values &nbsp;is taken from the property, and the result is stored in two lists, one for the <code>ComboBox </code>display and one for the actual <code>Enum </code>values available in the property. &nbsp;The display are hooked up, and an event is attached to the selection changes of the <code>ComboBox</code>. And that&#39;s ALL FOLKS, it now works simply by typing in the code in XAML:</p>

<pre lang="xml">
        &lt;ComboBox Name=&quot;cmbEnum&quot; 
&nbsp;                 local:EnumToComboBoxBinding.EnumItemsSource=&quot;{Binding TheEnumProperty}&quot;
                  ...
  /&gt;</pre>

<div>and I employed a little trick to just test the single class by setting the <code>DataContext </code>of the <code>ComboBox</code>:</div>

<pre lang="vb.net">
    Dim TheShowCaseClass As New TheComboBoxShowCase

    Private Sub Window_Loaded(sender As Object, e As RoutedEventArgs)

         ...

        cmbEnum.DataContext = TheShowCaseClass</pre>

<p>This little trick should make it easy to reuse it in all cases, or simply expand it with the additional functions you need.</p>

<h2>The searchable TreeView</h2>

<p>I was browsing through some articles about <code>TreeView&#39;s </code>when I came across this article called:</p>

<ul>
	<li>&nbsp;<a href="http://www.codeproject.com/Articles/718022/Searchable-WPF-TreeView">Searchable WPF TreeView</a>&nbsp;by Fredrik Bornander&nbsp;</li>
</ul>

<p>I absolutely loved the functionality of the filter based search he provided, and my first thought was:I like to add this in my projects as well. But that wasn&#39;t so easy (well that isn&#39;t quite true, but you would have to do quite a bit of work in order to make it happen in your case as well) if you hadn&#39;t already thought about it from the start. I decided to ditch the fancy search box he had, I had no need for it. I just needed a search based on changes in a <code>TextBox</code>, and the other stuff could easily be implemented as an afterthought, if the search field was up and running already.</p>

<p>So I started working on the helper class for the attached property that I wanted to create. It was clear to me that I needed to store each item in the <code>TreeView </code>with the children element. I also needed the underlying class that was binded to each of the <code>TreeViewItems</code>, and finally the search functionality that Fredrik had provided.</p>

<p>The class also needed to contain the reflection based search through properties of the type string. The helper class ended up looking like this, with pointers to the real values:</p>

<pre lang="vb.net">
Public Class TreeViewHelperClass

    Private pCurrentTreeViewItem As TreeViewItem
    Public Property CurrentTreeViewItem() As TreeViewItem
        Get
            Return pCurrentTreeViewItem
        End Get
        Set(ByVal value As TreeViewItem)
            pCurrentTreeViewItem = value
        End Set
    End Property

    Private pBindedClass As Object
    Public Property BindedClass() As Object
        Get
            Return pBindedClass
        End Get
        Set(ByVal value As Object)
            pBindedClass = value
        End Set
    End Property

    Private pChildren As New List(Of TreeViewHelperClass)
    Public Property Children() As List(Of TreeViewHelperClass)
        Get
            Return pChildren
        End Get
        Set(ByVal value As List(Of TreeViewHelperClass))
            pChildren = value
        End Set
    End Property

    Private Function FindString(obj As Object, ByVal SearchString As String) As Boolean

        If String.IsNullOrEmpty(SearchString) Then
            Return True
        End If

        If obj Is Nothing Then Return True

        For Each p As System.Reflection.PropertyInfo In obj.GetType().GetProperties()
            If p.PropertyType = GetType(String) Then
                Dim value As String = p.GetValue(obj)
                If value.ToLower.Contains(SearchString.ToLower) Then
                    Return True
                End If
            End If
        Next

        Return False
    End Function

    Private expanded As Boolean
    Private match As Boolean = True

    Private Function IsCriteriaMatched(criteria As String) As Boolean
        Return FindString(BindedClass, criteria)
    End Function

    Public Sub ApplyCriteria(criteria As String, ancestors As Stack(Of TreeViewHelperClass))
        If IsCriteriaMatched(criteria) Then
            IsMatch = True
            For Each ancestor In ancestors
                ancestor.IsMatch = True
            Next
        Else
            IsMatch = False
        End If

        ancestors.Push(Me) &#39; and then just touch me
        For Each child In Children
            child.ApplyCriteria(criteria, ancestors)
        Next
        ancestors.Pop()
    End Sub

    Public Property IsMatch() As Boolean
        Get
            Return match
        End Get
        Set(value As Boolean)
            If match = value Then Return

            match = value
            If CurrentTreeViewItem IsNot Nothing Then
                If match Then
                    CurrentTreeViewItem.Visibility = Visibility.Visible
                Else
                    CurrentTreeViewItem.Visibility = Visibility.Collapsed
                End If
            End If

            OnPropertyChanged(&quot;IsMatch&quot;)
        End Set
    End Property

    Public ReadOnly Property IsLeaf() As Boolean
        Get
            Return Not Children.Any()
        End Get
    End Property
End Class</pre>

<p>As you see from the class the filter uses <code>Visibility </code>changes in order to clear items that doesn&#39;t match. It is also a top down search, where all the parent <code>TreeViewItems </code>are found by iterating through the stack of previous elements.</p>

<p>The criteria for a match is now only matched by looking at <code>String </code>elements in the binded class, but it can easily be expanded to search for particular values of properties by simply alter the search function a little. Assuming that you are looking for an item that has the property name <code>id </code>and the value <code>45</code>, then you could simply type in <code>id==45</code> into the search string:</p>

<pre lang="vb.net">
        Dim PropertyName As String
        Dim ValueOfProperty As String

        PropertyName = SearchString.Split(&quot;==&quot;)(0)
        ValueOfProperty = SearchString.Split(&quot;==&quot;)(1)
   

        For Each p As System.Reflection.PropertyInfo In obj.GetType().GetProperties()
            If p.CanRead Then

                    If p.Name.ToLower = PropertyName.ToLower Then

                        Dim t As Type = If(Nullable.GetUnderlyingType(p.PropertyType), p.PropertyType)
                        Dim safeValue As Object = If((ValueOfProperty Is Nothing), Nothing, Convert.ChangeType(ValueOfProperty, t))

                        &#39;Get the value
                        Dim f = p.GetValue(obj)

                        &#39;Its the same type
                        If safeValue IsNot Nothing Then
                            If f = safeValue Then
                                Return True
                            Else
                                Return False
                            End If
                        Else
                            &#39; If you end up here you have entered the wrong element type of the property
                        End If
                    End If
                Next
            End If
        Next</pre>

<p>That was the easy part of the searchable <code>TreeView</code>, now we need to write the Attached dependency property and get the all the <code>TreeViewItems </code>and underlying classes. It was pretty clear that we needed a <code>SearchString </code>as the Attached dependency property, and that we need to have a new search every time the property changed.</p>

<p>The more difficult issue her is making sure that all the elements in the <code>TreeView </code>is visible and that they are all drawn up when we try to populate the items into the helper class. Here <a href="http://www.zagstudio.com/blog/493#.VVhtD7mqpBd">Bea Stollniz&#39;s blog</a> was a big help, so I implemented the function below, and now I could be fairly certain that all the elements was visible and expanded.</p>

<pre lang="vb.net">
            ApplyActionToAllTreeViewItems(Sub(itemsControl)
                                              itemsControl.IsExpanded = True
                                              itemsControl.Visibility = Visibility.Visible
                                              DispatcherHelper.WaitForPriority(DispatcherPriority.ContextIdle)
                                          End Sub, TreeViewControl)</pre>

<p>An <a href="https://msdn.microsoft.com/en-us/library/ff407130%28v=vs.110%29.aspx">MSDN</a> article about finding an item in the TreeView also explains (indirectly?) how to ensure that the element is populated.&nbsp;</p>

<p>To populate the items into the helper class, I found a <a href="https://social.msdn.microsoft.com/Forums/vstudio/en-US/a2988ae8-e7b8-4a62-a34f-b851aaf13886/windows-presentation-foundation-faq?forum=wpf#expand_treeview">MSDN FAQ</a> that was much more linear in getting the elements.</p>

<pre lang="vb.net">
    Private Shared Sub CreateInternalViewModelFilter(parentContainer As ItemsControl, ByRef ParentTreeItem As TreeViewHelperClass)

        For Each item As [Object] In parentContainer.Items
            Dim TreeViewItemHelperContainer As New TreeViewHelperClass()

            TreeViewItemHelperContainer.BindedClass = item
            Dim currentContainer As TreeViewItem = TryCast(parentContainer.ItemContainerGenerator.ContainerFromItem(item), TreeViewItem)
            TreeViewItemHelperContainer.CurrentTreeViewItem = currentContainer
            ParentTreeItem.Children.Add(TreeViewItemHelperContainer)

            If currentContainer IsNot Nothing AndAlso currentContainer.Items.Count &gt; 0 Then

                If currentContainer.ItemContainerGenerator.Status &lt;&gt; GeneratorStatus.ContainersGenerated Then

                    &#39; This indicates that the TreeView isn&#39;t fully created yet. 
                    &#39; That means that the code should not have reached this point 

                    &#39; If the sub containers of current item is not ready, we need to wait until 
                    &#39; they are generated. 
                    AddHandler currentContainer.ItemContainerGenerator.StatusChanged, Sub()
                                                                                          CreateInternalViewModelFilter(currentContainer, TreeViewItemHelperContainer)
                                                                                      End Sub
                Else
                    &#39; If the sub containers of current item is ready, we can directly go to the next 
                    &#39; iteration to expand them. 
                    CreateInternalViewModelFilter(currentContainer, TreeViewItemHelperContainer)
                End If

            End If
        Next
    End Sub</pre>

<p>The only thing left then was to run the actual filter (or search):</p>

<pre lang="vb.net">
            &#39;The first instance is a dummy that is not connected to the TreeView, but can initiate the Search
            TreeViewHelper.Item(0).ApplyCriteria(TempSearchString, New Stack(Of TreeViewHelperClass))</pre>

<h2>The Windows Form style TreeView for WPF</h2>

<p>This TreeView started out with a style generated by <a href="https://social.msdn.microsoft.com/Forums/vstudio/en-US/30cb182c-9419-40bd-946e-87971515fb95/show-treeview-nodes-connected-with-dotted-lines?forum=wpf#f1462086-90ce-4357-b0b5-783ab8aeda29">Niel Kronlage from Microsoft</a> that drew lines and added a ToggleButton for the expand and retract child TreeViewItems. He had also implemented a <code>ValueConverter </code>to get the last item, as a means to stop drawing the lines.</p>

<p>This wored well for a static <code>TreeView </code>that didn&#39;t add any new items. As there was no way of the TreeVeiw updating it&#39;s render, Alex P. (<a href="https://social.msdn.microsoft.com/Forums/vstudio/en-US/30cb182c-9419-40bd-946e-87971515fb95/show-treeview-nodes-connected-with-dotted-lines?forum=wpf#f1462086-90ce-4357-b0b5-783ab8aeda29">in the same thread</a>) created and Attatched Property and added an eventhandler in the constructor, so that changes in the collection would force the UI to update.</p>

<p>Then we have the last addition (before me) which was made by TuyenTk and published here on CodeProject as a Tip: &nbsp;<a href="http://www.codeproject.com/Tips/673071/WPF-TreeView-with-WinForms-Style-Fomat">WPF TreeView with WinForms Style Fomat</a>. He made some style changes but missed the Attatched property that updated the UI once you added new TreeViewITems.</p>

<p>Once I had implemented all the different parts from the others, I ued it when filtering/searching my <code>TreeView</code>. It turen out that it didnt go so well, as the UI didn&#39;t know about the collapsed items. I needed to attach and event on Visibility changed.</p>

<p>I also moved things around a bit, to make the re-usability better, all you have to do now is to merge the directories that holds the style in Application:</p>

<pre lang="xml">
    &lt;Application.Resources&gt;
        &lt;ResourceDictionary&gt;
            &lt;ResourceDictionary.MergedDictionaries&gt;
                &lt;ResourceDictionary Source=&quot;/WinFormStyleTreeView/ExpandCollapseToggleStyle.xaml&quot;/&gt;
                &lt;ResourceDictionary Source=&quot;/WinFormStyleTreeView/WinFormStyle.xaml&quot;/&gt;
            &lt;/ResourceDictionary.MergedDictionaries&gt;
        &lt;/ResourceDictionary&gt;
    &lt;/Application.Resources&gt;</pre>

<p>And be sure to add the <code>ExpandCollapseToggleStyle </code>first as it is used by the <code>WinFormStyle</code>. If you change it you will get a rather strange sounding error. The style can now be implemented on any of your <code>TreeView&#39;s </code>separately like this:</p>

<pre lang="xml">
        &lt;TreeView&gt;
            &lt;TreeView.Resources&gt;
                &lt;Style TargetType=&quot;{x:Type TreeViewItem}&quot; BasedOn=&quot;{StaticResource WinFormTreeView}&quot;/&gt;

                ...

            &lt;/TreeView.Resources&gt;
        &lt;/TreeView&gt;</pre>

<p>In the attached class, were our attached properties lives, we need to have a value that will indicate if it has any items below itself. If so the property, <code>IsLast </code>is set to false, otherwise it&#39;s set to true. If this isn&#39;t done, it sill simply draw the lines until it reaches the bottom <code>TreeViewItem </code>in the control. So the <code>IsLast </code>Dependency Property is set up:</p>

<pre lang="vb.net">
    Public Shared IsLastOneProperty As DependencyProperty = DependencyProperty.RegisterAttached(&quot;IsLastOne&quot;, GetType(Boolean), GetType(TVIExtender))

    Public Shared Function GetIsLastOne(sender As DependencyObject) As Boolean
        Return CBool(sender.GetValue(IsLastOneProperty))
    End Function
    Public Shared Sub SetIsLastOne(sender As DependencyObject, isLastOne As Boolean)
        sender.SetValue(IsLastOneProperty, isLastOne)
    End Sub</pre>

<p>However, we need this event to fire if the collection is changed, and the best way to hook the event up would be to place it in the constructor <code>Sub New()</code>. The common way to do this is to attach a boolean dependency property called IsUsed or something similar. This should only be set once, when the object holding the attached dependency property is initiated, and you can have a <code>CallBackFunction </code>to set up initial bindings on item created.</p>

<p>Alex P. does this trough reacting to changes in the <code>UseExtenderProperty</code>, &nbsp;and initiates a new <code>TVIExtender </code>with the <code>TreeViewItem </code>as it&#39;s argument:</p>

<pre lang="vb.net">
    Private _item As TreeViewItem


    Public Sub New(item As TreeViewItem)
        _item = item

        Dim ic As ItemsControl = ItemsControl.ItemsControlFromItemContainer(_item)
        AddHandler ic.ItemContainerGenerator.ItemsChanged, AddressOf OnItemsChangedItemContainerGenerator

        _item.SetValue(IsLastOneProperty, ic.ItemContainerGenerator.IndexFromContainer(_item) = ic.Items.Count - 1)
    End Sub</pre>

<p>The code works simply by getting the TreeViewItem that holds the newly constructed child (also a <code>TreeViewItem</code>) using the <code>ItemsContol.ItemsControlFromItemContainer</code>. Then it adds a handler to the item changed, that fire each time the collection changes, and If the current item is the last in the collection, the <code>IsLastproperty </code>is set to true. And if the collection is changed we are back to the same setting again:</p>

<pre lang="vb.net">
    Private Sub OnItemsChangedItemContainerGenerator(sender As Object, e As ItemsChangedEventArgs)
        Dim ic As ItemsControl = ItemsControl.ItemsControlFromItemContainer(_item)

        If ic IsNot Nothing Then
            _item.SetValue(IsLastOneProperty, ic.ItemContainerGenerator.IndexFromContainer(_item) = ic.Items.Count - 1)
        End If
    End Sub</pre>

<p>So far I have just explained what Alex P. has done in his implementation of the Attached DependencyProperty, and as it is now it won&#39;t react correctly if one changes the <code>Visiblility </code>of a <code>TreeViewItem</code>. We must then recalculate the <code>IsLast </code>property if the visibility value changes from <code>Visible </code>or <code>Hidden </code>(they will have the TreeView rendered the same way) to <code>Collapsed</code>. To attatch an event to changes in a DependencyProperty I used the&nbsp;<a href="https://msdn.microsoft.com/en-us/library/system.componentmodel.dependencypropertydescriptor%28v=vs.110%29.aspx">DependencyPropertyDescriptor&nbsp;</a>class.</p>

<pre lang="vb.net">
   Private Shared VisibilityDescriptor As DependencyPropertyDescriptor = DependencyPropertyDescriptor.FromProperty(TreeViewItem.VisibilityProperty, GetType(TreeViewItem))</pre>

<p>I added code to bind the VisibilityChange to a sub, in the constructor:</p>

<pre lang="vb.net">
    Public Sub New(item As TreeViewItem)
         ...

        VisibilityDescriptor.AddValueChanged(_item, AddressOf VisibilityChanged)

        ...       

    End Sub</pre>

<p>This would now run the sub <code>VisibilityChanged</code> each time, a word of warning however. Each time you add an event or a Descriptor don&#39;t forget to mop up after you are done with it.</p>

<pre lang="vb.net">
    Private Sub Detach()
        If _item IsNot Nothing Then
            Dim ic As ItemsControl = ItemsControl.ItemsControlFromItemContainer(_item)
            If ic IsNot Nothing Then
                RemoveHandler ic.ItemContainerGenerator.ItemsChanged, AddressOf OnItemsChangedItemContainerGenerator
            End If

            VisibilityDescriptor.RemoveValueChanged(_item, AddressOf VisibilityChanged)
        End If
    End Sub</pre>

<p>&nbsp;Now that we have a sub that is run each time the visibility changes we start off with writing the code:</p>

<pre lang="vb.net">
    Private Sub VisibilityChanged(sender As Object, e As EventArgs)
        If TypeOf (sender) Is TreeView Then
            Exit Sub
        End If

        If DirectCast(_item, ItemsControl).Visibility = Visibility.Collapsed Then
            Dim ic As ItemsControl = ItemsControl.ItemsControlFromItemContainer(_item)
            Dim Index As Integer = ic.ItemContainerGenerator.IndexFromContainer(_item)

            If Index &lt;&gt; 0 And _item.GetValue(IsLastOneProperty) Then
                DirectCast(ic.ItemContainerGenerator.ContainerFromIndex(Index - 1), TreeViewItem).SetValue(IsLastOneProperty, True)
            End If
        Else
            Dim ic As ItemsControl = ItemsControl.ItemsControlFromItemContainer(_item)
            Dim Index As Integer = ic.ItemContainerGenerator.IndexFromContainer(_item)

            If Index &lt;&gt; 0 Then
                DirectCast(ic.ItemContainerGenerator.ContainerFromIndex(Index - 1), TreeViewItem).SetValue(IsLastOneProperty, False)
            End If
        End If
    End Sub</pre>

<p>The code in itself is actually really simple, if the property IsLast is true, set the IsLast to true on the previous element, unless the current element is the only one in the collection. The other way around you can just set the previous element to false regardless of what it&#39;s value was. And that is all you need to have a functioning <code>TreeView </code>when items are collapsible.</p>

<p>There is one more issue with the use of the <a href="http://stackoverflow.com/questions/23682232/how-can-i-fix-the-dependencypropertydescriptor-addvaluechanged-memory-leak-on-at">DependencyPropertyDescriptor </a>regards to detaching the event. I found <a href="https://agsmith.wordpress.com/2008/04/07/propertydescriptor-addvaluechanged-alternative/">Andrew Smith&#39;s</a>&nbsp;blog entery from 2008, and translated that code to VB, and its in the Unused folder. However, 2008 is a long time ago, and articles on <a href="https://msdn.microsoft.com/en-us/magazine/cc794276.aspx#id0070111">MSDN</a> after it was published, still used the original method, so I left it as it is. If anyone know if it has been fixed, or if it&#39;s still a problem I&#39;d really like to know about it.&nbsp;</p>

<h2>Adding a pixel shader magnifier</h2>

<p>Several years ago I remember seeing a Silverlight article that had the coolest looking magnifier glass I had ever seen, and it was create using pixel shaders. I didnt have the time to dig into the code so I just bookmarked the site and forgot about it all. Then as I did reasearch to this article, I came across this article:&nbsp;<a href="http://www.codeproject.com/Articles/69083/WPF-Parent-Window-Shading-Using-Pixel-Shaders">WPF Parent Window Shading Using Pixel Shaders</a>, and I started thinking. Do I still have the link to that article with the cool magnifier? I did:&nbsp;<a href="http://www.silverlightshow.net/items/Behaviors-and-Triggers-in-Silverlight-3.aspx">Behaviors and Triggers in Silverlight</a>, so let&#39;s get going and implement it now.</p>

<p>The source coe, you know the .fx file, that the DirectX compiler makes into a .ps file (you can read more about the compiling of these <a href="http://blogs.msdn.com/b/chuckw/archive/2012/05/07/hlsl-fxc-and-d3dcompile.aspx">here</a>. I will go through the bare minimum for you to start to understand what you need to take into account just to make them work, for a more detailed review and some cool tools and program see these links:</p>

<ul>
	<li><a href="http://www.codeproject.com/Articles/71617/Getting-Started-with-Shader-Effects-in-WPF">Getting Started with Shader Effects in WPF</a></li>
	<li><a href="http://blog.wpfwonderland.com/2008/10/08/shazzam-wpf-pixel-shader-effect-testing-tool-now-available/">Shazzam &ndash; WPF Pixel Shader Effect Testing Tool</a></li>
	<li><a href="http://wpfshadergenerator.codeplex.com/">WPF ShaderEffect Generator</a></li>
</ul>

<p>The (complete!) file looke like this:</p>

<pre lang="C++">
float2 center : register(C0);
float inner_radius: register(C2);
float magnification : register(c3);
float outer_radius : register(c4);

SamplerState  Input : register(S0);

float4 main( float2 uv : TEXCOORD) : COLOR
{
    float2 center_to_pixel = uv - center; // vector from center to pixel  
    float distance = length(center_to_pixel);
    float4 color;
    float2 sample_point;
    
    if(distance &lt; outer_radius)
    {
      if( distance &lt; inner_radius )
      {
         sample_point = center + (center_to_pixel / magnification);
      }
      else
      {
          float radius_diff = outer_radius - inner_radius;
          float ratio = (distance - inner_radius ) / radius_diff; // 0 == inner radius, 1 == outer_radius
          ratio = ratio * 3.14159; //  -pi/2 .. pi/2          
          float adjusted_ratio = cos( ratio );  // -1 .. 1
          adjusted_ratio = adjusted_ratio + 1;   // 0 .. 2
          adjusted_ratio = adjusted_ratio / 2;   // 0 .. 1
       
          sample_point = ( (center + (center_to_pixel / magnification) ) * (  adjusted_ratio)) + ( uv * ( 1 - adjusted_ratio) );
      }
    }
    else
    {
       sample_point = uv;
    }

    return tex2D( Input, sample_point );    
}</pre>

<p>At the very start you see 4 input parameters into the code, named C0,C2,C3 and C4, and these will link up to their separate Dependency Properties. The variable marked Input is an&nbsp;<span style="background-color: rgba(255, 255, 255, 1)">&quot;image register&quot; that is also linked to a Dependency Property. The dependency properties look like this:</span></p>

<pre lang="vb.net">
    Public Shared ReadOnly CenterProperty As DependencyProperty =
        DependencyProperty.Register(&quot;Center&quot;,
                                    GetType(Point),
                                    GetType(Magnifier),
                                    New PropertyMetadata(New Point(0.5, 0.5),
                                                         PixelShaderConstantCallback(0)))

    Public Shared ReadOnly InnerRadiusProperty As DependencyProperty =
        DependencyProperty.Register(&quot;InnerRadius&quot;,
                                    GetType(Double),
                                    GetType(Magnifier),
                                    New PropertyMetadata(0.2, PixelShaderConstantCallback(2)))</pre>

<p>You see that the&nbsp;</p>

<pre lang="vb.net">
PixelShaderConstantCallback</pre>

<p>has the same value in the functon that the C variables had in the fx file. In the same file (Magnifier) we also reset the PixelShader property (which is inhereted from the ShaderEffect class in the ShaderEffectBase):</p>

<pre lang="vb.net">
Public MustInherit Class ShaderEffectBase
    Inherits ShaderEffect</pre>

<p>The shadereffects.PixelShader is just an empty pointer so it needs a new instance of PixelShader:&nbsp;</p>

<pre lang="vb.net">
    Sub New()
        PixelShader = New PixelShader
        PixelShader.UriSource = New Uri(AppDomain.CurrentDomain.BaseDirectory &amp; &quot;\ShaderSourceFiles\Magnifier.ps&quot;)

        Me.UpdateShaderValue(CenterProperty)
        Me.UpdateShaderValue(InnerRadiusProperty)
        Me.UpdateShaderValue(OuterRadiusProperty)
        Me.UpdateShaderValue(MagnificationProperty)
    End Sub</pre>

<p>The UriSource is a real pain, it needs to find the compiled file *.ps at that spesific location, otherwise your program will crash.&nbsp;</p>

<p>Now its just the class with the Attached Dependency Property left, and I will initiate it (as per usual) with a bool value set to true:</p>

<pre lang="vb.net">
    Public Shared MagnifyProperty As DependencyProperty =
        DependencyProperty.RegisterAttached(&quot;Magnify&quot;,
                                            GetType(Boolean),
                                            GetType(MagnifierOverBehavior),
                                            New FrameworkPropertyMetadata(AddressOf MagnifiedChanged))</pre>

<p>And the callback to MagnifiedChanged we attach and detach the handlers used:</p>

<pre lang="vb.net">
    Public Shared Sub MagnifiedChanged(sender As DependencyObject, e As DependencyPropertyChangedEventArgs)
        AssociatedObject = TryCast(sender, FrameworkElement)
        If AssociatedObject IsNot Nothing Then
            If e.NewValue Then
                OnAttached()
            Else
                OnDetaching()
            End If
        End If
    End Sub</pre>

<p>The two classes are exact opposite of one another:</p>

<pre lang="vb.net">
    Private Shared Sub OnAttached()
        AddHandler AssociatedObject.MouseEnter, AddressOf AssociatedObject_MouseEnter
        AddHandler AssociatedObject.MouseLeave, AddressOf AssociatedObject_MouseLeave
        AddHandler AssociatedObject.MouseMove, AddressOf AssociatedObject_MouseMove
        AssociatedObject.Effect = magnifier
    End Sub

    Private Shared Sub OnDetaching()
        RemoveHandler AssociatedObject.MouseEnter, AddressOf AssociatedObject_MouseEnter
        RemoveHandler AssociatedObject.MouseLeave, AddressOf AssociatedObject_MouseLeave
        RemoveHandler AssociatedObject.MouseMove, AddressOf AssociatedObject_MouseMove
        AssociatedObject.Effect = Nothing
    End Sub</pre>

<p>This adds up to reactions in the mouse move section, that initiates a StoryBoard to move the Magnigication:</p>

<pre lang="vb.net">
    Private Shared Sub AssociatedObject_MouseMove(sender As Object, e As MouseEventArgs)

        TryCast(AssociatedObject.Effect, Magnifier).Center = e.GetPosition(AssociatedObject)

        Dim mousePosition As Point = e.GetPosition(AssociatedObject)
        mousePosition.X /= AssociatedObject.ActualWidth
        mousePosition.Y /= AssociatedObject.ActualHeight
        magnifier.Center = mousePosition

        Dim zoomInStoryboard As New Storyboard()
        Dim zoomInAnimation As New DoubleAnimation()
        zoomInAnimation.[To] = magnifier.Magnification
        zoomInAnimation.Duration = TimeSpan.FromSeconds(0.5)
        Storyboard.SetTarget(zoomInAnimation, AssociatedObject.Effect)
        Storyboard.SetTargetProperty(zoomInAnimation, New PropertyPath(magnifier.MagnificationProperty))
        zoomInAnimation.FillBehavior = FillBehavior.HoldEnd
        zoomInStoryboard.Children.Add(zoomInAnimation)
        zoomInStoryboard.Begin()
    End Sub</pre>

<p>The magnifier will magnify anyting that is a FrameworkElement, and can be turen on and off by a boolean values.</p>
