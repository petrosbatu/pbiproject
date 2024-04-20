# Дашборд продаж и возвратов в Power BI

### Dashboard Link : 

## Контекст

В сгенерированном датасете имеются данные о продажах некоторой фирмы, а так же по возвратам заказов.

#### Содержание источника данных:

1. Таблица Orders: 
- номер заказа
- данные о клиенте
- класс доставки (в тот же день/первый класс/второй класс/стандартный класс)
- приоритет заказа
- адрес доставки
- даты принятия заказа и его доставки
- прибыль заказа и кол-во товара

2. Таблица Returns:
- статус возвращения (Yes/No)
- номер заказа
- регион заказа

3. Таблица People - не используется

## Постановка задачи

Создать отчёт, который выявляет по каким товарам, способам доставки и другим категориям данных чаще происходят возвраты. Отчёт должен выявить, какие товары сильнее всего влияют на убытки, связанные с возвратами, и на какие из продуктов следует обратить внимание в первую очередь, чтобы снизить частоту возвратов.


## Шаги создания и описание используемых метрик 

#### Преобразования в Power Query

* Шаг 1 : Загрузка данных из .xslx файла (таблицы **Orders** и **Returns**, далее переименованы в **FactOrders** и **FactReturns**)

* Шаг 2 : Дублируем таблицу FacrOrders и извлекаем из неё столбцы с информацией о покупателях (**CustomerName**, **Segment**). **CustomerID** не используется см. шаг 3. Называем таблицу DimCustomers

* Шаг 3 : В исходных данных у одного покупателя имеется несколько **Customer ID**. 

![multiple_ids](https://github.com/petrosbatu/pbiproject/blob/main/images/Multiple_IDs.jpg?raw=true)

Группируем клиентов по **Customer Name**, **Segment** (категория клиента). Добавляем временный столбец с индексами с первым значением и инкрементом равными 1. Затем с помощью следующей формулы присваем новый **CustomerID**, схожий по своему формату со старыми идентификаторами (первые буквы имени и фамилии и числовое значение c фиксированной длиной и ведущими нулями через дефис).

```powerquery
    Text.Start(Text.BeforeDelimiter([Customer Name], " "), 1) & 
    Text.Start(Text.AfterDelimiter([Customer Name], " "), 1) &
    "-" & Number.ToText([Index], "000000")
```

Получаем таблицу следующего вида:

![dim_customers](https://github.com/petrosbatu/pbiproject/blob/main/images/DimCustomers.jpg?raw=true)

При этом старые значения идентификаторов хранятся в источнике данных и при необходимости могут быть использованны.

* Шаг 4 : Дублируем таблицу **FactOrders** и извлекаем из неё столбцы с информацией о товарах. Сохраняем только уникальные строки (каждый товар мог быть заказан многократно). Получаем таблицу **DimProducts**.

* Шаг 5 : К таблице FactOrders присоединяем таблицу DimCustomers по CustomerName. Из правой таблицы раскрываем столбец **CustomerID**.

* Шаг 6 : Из таблицы заказов убираем те столбцы, которые были перенесены в таблицу измерений (кроме идентификаторов). Также удаляем старый **Customer ID** и другие поля, которые не будут использоватася в дальнейших расчётах и визуализациях.

* Шаг 7 : В таблице **FactReturns** убираем столбец **Returned**, так как в нём все значения равны "Yes" (иначе говоря, в таблице есть записи только о тех заказах, которые были возвращены).

![returned](https://github.com/petrosbatu/pbiproject/blob/main/images/returned.jpg?raw=true)

* Шаг 8 : применяем преобразования power query.

#### Расчёты DAX

* Шаг 9 : Создаём календарь **DimDates** с помощью формулы:

```DAX
DimDates = 
ADDCOLUMNS(
    CALENDARAUTO(),
    "Year", YEAR([Date]),
    "MonthName", FORMAT([Date], "MMMM"),
    "MonthNumber", MONTH([Date]),
    "Day", DAY([Date])
)
```
Устанавливаем сортировку названий месяца по его номеру.

* Шаг 10 : Устанавливаем связи между таблицами.  

* Шаг 10 : Добавляем таблицу мер и создаём две меры. 
1. **Return Rate** - частота возвратов. Число заказов с возвратами        
```DAX
Return Rate = COUNTROWS(FactReturns) / COUNTROWS(DISTINCT(FactOrders[Order ID]))
```
- Step 9 : Two card visuals were added to the canvas, one representing average departure delay in minutes & other representing average arrival delay in minutes.
           Using visual level filter from the filters pane, basic filtering was used & null values were unselected for consideration into average calculation.
           
           Although, by default, while calculating average, blank values are ignored.
- Step 10 : A bar chart was also added to the report design area representing the number of satisfied & neutral/unsatisfied customers. While creating this visual, field named "Gender" was also added to the Legends bucket, thus number of customers are also seggregated according the gender. 
- Step 11 : Ratings Visual was used to represent different ratings mentioned below,

  (a) Baggage Handling

  (b) Check-in Services
  
  (c) Cleanliness
  
  (d) Ease of online booking
  
  (e) Food & Drink
  
  (f) In-flight Entertainment

  (g) In-flight Service
  
  (h) In-flight wifi service
  
  (i) Leg Room service
  
  (j) On-board service
  
  (k) Online boarding
  
  (l) Seat comfort
  
  (m) Departure & arrival time convenience
  
In our dataset, Some parameters were assigned value 0, representing those parameters are not applicable for some customers.

All these values have been ignored while calculating average rating for each of the parameters mentioned above.

- Step 12 : In the report view, under the insert tab, two text boxes were added to the canvas, in one of them name of the airlines was mentioned & in the other one company's tagline was written.
- Step 13 : In the report view, under the insert tab, using shapes option from elements group a rectangle was inserted & similarly using image option company's logo was added to the report design area. 
- Step 14 : Calculated column was created in which, customers were grouped into various age groups.

for creating new column following DAX expression was written;
       
        Age Group = 
        
        if(airline_passenger_satisfaction[Age]<=25, "0-25 (25 included)",
        
        if(airline_passenger_satisfaction[Age]<=50, "25-50 (50 included)",
        
        if(airline_passenger_satisfaction[Age]<=75, "50-75 (75 included)",
        
        "75-100 (100 included)")))
        
Snap of new calculated column ,

![Snap_1](https://user-images.githubusercontent.com/102996550/174089602-ab834a6b-62ce-4b62-8922-a1d241ec240e.jpg)

        
- Step 15 : New measure was created to find total count of customers.

Following DAX expression was written for the same,
        
        Count of Customers = COUNT(airline_passenger_satisfaction[ID])
        
A card visual was used to represent count of customers.

![Snap_Count](https://user-images.githubusercontent.com/102996550/174090154-424dc1a4-3ff7-41f8-9617-17a2fb205825.jpg)

        
 - Step 16 : New measure was created to find  % of customers,
 
 Following DAX expression was written to find % of customers,
 
         % Customers = (DIVIDE(airline_passenger_satisfaction[Count of Customers], 129880)*100)
 
 A card visual was used to represent this perecntage.
 
 Snap of % of customers who preferred business class
 
 ![Snap_Percentage](https://user-images.githubusercontent.com/102996550/174090653-da02feb4-4775-4a95-affb-a211ca985d07.jpg)

 
 - Step 17 : New measure was created to calculate total distance travelled by flights & a card visual was used to represent total distance.
 
 Following DAX expression was written to find total distance,
 
         Total Distance Travelled = SUM(airline_passenger_satisfaction[Flight Distance])
    
 A card visual was used to represent this total distance.
 
 
 ![Snap_3](https://user-images.githubusercontent.com/102996550/174091618-bf770d6c-34c6-44d4-9f5e-49583a6d5f68.jpg)
 
 - Step 18 : The report was then published to Power BI Service.
 
 
![Publish_Message](https://user-images.githubusercontent.com/102996550/174094520-3a845196-97e6-4d44-8760-34a64abc3e77.jpg)

# Snapshot of Dashboard (Power BI Service)

![dashboard_snapo](https://user-images.githubusercontent.com/102996550/174096257-11f1aae5-203d-44fc-bfca-25d37faf3237.jpg)

 
 # Report Snapshot (Power BI DESKTOP)

 
![Dashboard_upload](https://user-images.githubusercontent.com/102996550/174074051-4f08287a-0568-4fdf-8ac9-6762e0d8fa94.jpg)

# Insights

A single page report was created on Power BI Desktop & it was then published to Power BI Service.

Following inferences can be drawn from the dashboard;

### [1] Total Number of Customers = 129880

   Number of satisfied Customers (Male) = 28159 (21.68 %)

   Number of satisfied Customers (Female) = 28269 (21.76 %)

   Number of neutral/unsatisfied customers (Male) = 35822 (27.58 %)

   Number of neutral/unsatisfied customers (Female) = 37630 (28.97 %)


           thus, higher number of customers are neutral/unsatisfied.
           
### [2] Average Ratings

    a) Baggage Handling - 3.63/5
    b) Check-in Service - 3.31/5
    c) Cleanliness - 3.29/5
    d) Ease of online booking - 2.88/5
    e) Food & Drink - 3.21/5
    f) In-flight Entertainment - 3.36/5
    g) In-flight service - 3.64/5
    h) In-flight Wifi service - 2.81/5
    i) Leg room service - 3.37/5
    j) On-board service - 3.38/5
    k) Online boarding - 3.33/5
    l) Seat comfort - 3.44/5
    m) Departure & arrival convenience - 3.22/5
  
  while calculating average rating, null values have been ignored as they were not relevant for some customers. 
  
  These ratings will change if different visual filters will be applied.  
  
  ### [3] Average Delay 
  
      a) Average delay in arrival(minutes) - 15.09
      b) Average delay in departure(minutes) - 14.71
Average delay will change if different visual filters will be applied.

 ### [4] Some other insights
 
 ### Class
 
 1.1) 47.87 % customers travelled by Business class.
 
 1.2) 44.89 % customers travelled by Economy class.
 
 1.3) 7.25 % customers travelled by Economy plus class.
 
         thus, maximum customers travelled by Business class.
 
 ### Age Group
 
 2.1)  21.69 % customers belong to '0-25' age group.
 
 2.2)  52.44 % customers belong to '25-50' age group.
 
 2.3)  25.57 % customers belong to '50-75' age group.
 
 2.4)  0.31 % customers belong to '75-100' age group.
 
         thus, maximum customers belong to '25-50' age group.
         
### Customer Type

3.1) 18.31 % customers have customer type 'First time'.

3.2) 81.69 % customers have customer type 'returning'.
       
       thus, more customers have customer type 'returning'.

### Type of travel

4.1) 69.06 % customers have travel type 'Business'.

4.2) 30.94 % customers have travel type 'Personal'.

        thus, more customers have travel type 'Business'.
