# Walmart-Recruiting---Store-Sales-Forecasting

Link to the report on Wand&B - https://wandb.ai/final-project-ml/walmart-sales-prediction/reports/Walmart-Sales-Report--VmlldzoxMzUwNjA1MA

**შესავალი :**
მოცემული პროქტის მიზანი იყო Walmart-ის სხვადასხვა რეგიონში მდებარე 45 მაღაზიის გაყიდვების პროგნოზირება. პროექტზე მუშაობისას დავტესტეთ დროითი მწკრივების პროგნოზირებისთვის რამდენიმე მეთოდი. კერძოდ,  
* Deep Learning - N-BEATS, PatchTST, DLinear
* Tree-Based Models - LightGBM, XGBoost
* Classical Statistical Time-Series Models - ARIMA, SARIMA, SARIMAX, Prophet

გადმოგვეცემოდა შემდეგი ფაილები : 
train.csv მთავარი სატრენინგო ფაილი 5 სვეტით : Store, Dept, Date, Weekly_Sales (target), isHoliday. 
test.csv სატესტო ფაილი. იგივე რაც train.csv Weekly_Sales გარეშე. 
features.csv - დამატებითი მონაცემებისთვის, როგორიცაა ტემპერატურა, საწვავის ფასი, ფასდაკლების კვირეული, უმუშევრობისა და სამომხმარებლო ფასების ინდექსი. ასევე გადმოცემული გვქონდა 4 კონკრეტული დღესასწაულის თარიღები: Super Bowl, Labor Day, მადლიერების დღე და შობა. 

-----------------------------------------------------------------------------------------------------------------------------
**მონაცემების ანალიზი :** ავაგეთ სხვადასხვა პლოტი, რომ დაგვენახა ზოგადი ტრენდი, სეზონურობა, თუ რა მნიშვნელობა ჰქონდა გაყიდვებზე მაღაზიის ტიპსა თუ დეპარტამენტს, რამდენად განმსაზღვრელი იყო დღესასწაულის პერიოდი, რა კორელაცია იყო ფიჩერებს შორის. გამოვიყენეთ ავტოკორელაციისა და  ნაწილობრივი ავტოკორელაციის ფუნქციაც.

-----------------------------------------------------------------------------------------------------------------------------

**მონაცემთა ინჟინერია და მათი ანალიზი :** 
1) თარიღებიდან მივიღეთ ახალი ცვლადები: წელიწადი, თვე, კვირა, კვირის დღე, კვარტალი, თვის დასაწყისი/დასასრული, დღე თვეში.
2) იმისთვის, რომ სეზონურობა და პერიოდულობა უკეთ აღვიქვათ, თვე, კვირა და კვირის დღე გადავაქციეთ ციკლურ ცვლადებად სინუსისა და კოსინუსის ფუნქციებით.
1-ლი და 12-ე თვეები ერთმანეთთან ახლოს არიან, მაგრამ თუ მხოლოდ რიცხვით წარმოვადგენთ, მათ შორის დიდი განსხვავება გამოვა. ამიტომ, მათი sinus და cosinus ფუნქციებში გადატანა პრობლემას აგვარებს:
sin(2π ∗ value/max value)cos(2π ∗ value/max value). ასე მოდელი ხედავს, რომ დეკემბერი და იანვარი ერთმანეთს ემიჯნება, ისევე, როგორც კვირა და ორშაბათი.
3) დავამატეთ ინდიკატორები სუპერბოულის, შრომის დღის, მადლიერების დღისა და შობისთვის. თვრის დასაწყისი და დასასრული გამოვყავით, როგორც გარდამავალი პერიოდები, რადგან სწორედ ამ დღეებში ხშირად შეიმჩნევა გაყიდვების დინამიკაში მკვეთრი ცვლილებები. ამას ხელს უწყობს ისეთი ფაქტორები, როგორიცაა: ხელფასების გადახდები, მაღაზიების სეზონური ან თვიური ფასდაკლებები, ან მომხმარებელთა ფინანსური ჩვევები თვის დასაწყისსა და ბოლოს. ასევე შევქმენი HolidayProximity ცვლადი, რომელიც აჩვენებს არის თუ არა თარიღი დღესასწაულამდე ან მის შემდეგ 1-2 კვირის შუალედში. მაგალითად, შობის წინ და შემდეგ მომხმარებელი შეიძლება აქტიურობდეს ფასდაკლებებზე.
4) Lag Feature ანუ დაგვიანებული ცვლადი — ეს არის ტექნიკა, რომლის დროსაც დროის სერიის წინა მნიშვნელობა ინახება ახალი ცვლადის სახით და ემატება მოდელს, როგორც პროგნოზირებისთვის გამოსაყენებელი დამატებითი ინფორმაცია. მოდელი ამას ვერ გაიხსენებს, თუ წარსულის მონაცემებს ცალკე არ მივაწვდით. Lag Feature-ები სწორედ ამას აკეთებს — აძლევს მოდელს წინა დროის წერტილების მნიშვნელობებს. ასევე გამოვიყენეთ მოძრავი საშუალო, ე.წ "rolling mean", ეს არის ბოლო N პერიოდის საშუალო მნიშვნელობა.
5) გამოვითვალე მაღაზიის, დეპარტამენტის და მაღაზია+დეპარტამენტის მიხედვით გაყიდვების საშუალო, სტანდარტული გადახრა და მედიანა. მაღაზიის ან დეპარტამენტის დონეზე გაყიდვების სტატისტიკური სურათი საშუალებას აძლევს მოდელს ადვილად გაიგოს რომელ მაღაზიებს ან დეპარტამენტებს აქვთ შედარებით მაღალი ან არასტაბილური გაყიდვები.
6) ჩავამატეთ ფასდაკლებასთან დაკავშირებული ცვლადები (Total_MarkDown), აქტიური ფასდაკლებების რაოდენობა, მაქსიმალური ფასდაკლება და Markdown-ის ინტენსივობა (ფასდაკლებების ოდენობა მაღაზიის ზომის მიხედვით). ფასდაკებები მნიშვნელოვან გავლენას ახდენენ გაყიდვების ზრდაზე. მათი რაოდენობა, მასშტაბი და ინტენსივობა ხშირად პირდაპირ კავშირშია გაყიდვების მოცულობასთან.
7) ეკონომიკური ცვლადების სახით შევქმენით ახალი სვეტები, როგორიცაა CPI / უმუშევრობა, საწვავის ფასი CPI-თან მიმართებაში, და ნორმალიზებული მაჩვენებლები. მნიშვნელოვანი ეკონომიკური ცვლილებები ცალსახად მოქმედებენ გაყიდვებზე.

აღნიშნული ცვლადები მოდელების დატრენინგებისას გამოვიყენეთ სხვადასხვა კომბინაციით, ზოგიერთმა მოდელმა უკეთესი შედეგი აჩვენა მაშინ, როდესაც ყველა ცვლადი გამოყენებული იყო ტრენინგში, ზოგიერთისთვის კი ამოვარჩიეთ ისინი feature-importance -ის მიხედვით. ჩავამატეთ წონები დღესასწაულებისთვის (წონა: 5 დღესასწაულებზე და 1 — სხვა დღეებზე).
გავაკეთეთ StandardScaler-ით ნორმალიზაცია, იმ მოდელებისთვის, რომლებისთვისაც ნორმალიზაცია აუცილებელია. ახალა განვიხილივთ მოდელებს ინდივიდუალურად. 


-----------------------------------------------------------------------------------------------------------------------------
**ტესტირებული მოდელები :**

***Tree-Based Models*** 

1. **XGBoost (Extreme Gradient Boosting)** წარმოადგენს ensemble არქიტექტურას, სადაც მრავალი სუსტი decision tree-ის კომბინაციით იქმნება ძლიერი მოდელი.  
დროითი მონაცემებისთვის XGBoost-ის გამოყენებისას მოდელი თითოეულ დროით ეტაპზე პროგნოზირებად მნიშვნელობას Decision Tree-ების საშუალებით აფასებს. განსხვავებით ნეირონული ქსელებისგან, აქ სექვენსური დამოკიდებულება არ ინახება პირდაპირ, თუმცა სწორი ცვლადებით და წონების მექანიზმით შესაძლებელია დროითი ხასიათის ეფექტური დაჭერა.

საწყის ეტაპზე მონაცემთა გაწმენდის შემდეგ, ჰიპერპარამეტრები შევარჩიე დროით სერიებზე ეფექტური მუშაობისთვის, განსაკუთრებით ისეთი პერიოდის გათვალისწინებით, როგორიცაა დღესასწაულები და სეზონური პიკური გაყიდვები. შესაბამისად, holiday პერიოდებში მნიშვნელობებს 5-ჯერ მეტი წონა მივანიჭე, რათა მოდელს უკეთესად ესწავლა განსაკუთრებული დღეების გავლენა.

გამოყენებული  პარამეტრები:  
პარამეტრი	-   მნიშვნელობა  
max_depth  -	8  
learning_rate	 -  0.05  
n_estimators	-  2700  
early_stopping - 	50 rounds  
sampling_weight	-  5x (holiday)  

ჰიპერპარამეტრების ტესტირება  
სწავლის პროცესში სხვადასხვა ჰიპერპარამეტრების კომბინაცია გამოვცადე.  
კერძოდ:  
max_depth: 4, 6, 8, 10  
learning_rate: 0.01, 0.03, 0.05  
n_estimators: 1500, 2000, 2700, 3500  

ამ ცდებიდან საუკეთესო შედეგი ზემოთ მოყვანილი სეტინგით მივიღე, სადაც ბალანსი ეფექტურობასა და overfitting-ის რისკს შორის ყველაზე ოპტიმალური იყო. დავაგენერირე submission ფაილი უნახავ მონაცემებზე და kaggle - ზე ავტვირთე, სადაც დააფიქსირა 3040 wmae (~180 leaderboard-ზე).    
შედეგი: Training WMAE: 613.50, Validation WMAE: 1163.57. Overfitting ratio: 1.9x, რაც მსგავსი სირთულის მონაცემებისთვის მისაღები დონეა.

--------------------------------------------------------------------------------------------------------------

2. **LightGBM** წარმოადგენს ensemble არქიტექტურას - ბევრი სუსტი ხისგან საბოლოოდ ვიღებთ ძლიერ მოდელს. ხეები იგება თანმიმდევრულად და თითოეული ხე წინა ხის დაშვებულ შეცდომას ასწორებს.

XGBoost-გან განსხვავებით, ხეების დონეების მიხედვით გაზრდის ნაცვლად, LightGBM ზრდის მათ ფოთლების მიხედვით. შეუძლია _NaN მნიშვნელობების დამუშავება_, ისევე როგორც _კატეგორიული მონაცემების_ - one-hot encoding-ის გარეშე. მოდელს არ უჭირს მრავალ ფიჩერებთან მუშაობა - ტრენინგის დროს შეუძლია გაარკვიოს, რომელია მნიშვნელოვანი.  

თუმცა, LightGBM დროს ბუნებრივად ვერ აღიქვამს, ამიტომ მონაცემთა ინჟინერიაა საჭირო. 
დავამატეთ Lag ფიჩერები, ანუ weekly sales მნიშვნელობა გარკვეული N კვირის წინ. ეს მახასიათებლები მოდელს ეხმარება _მოკლევადიანი პატერნების_ შესწავლაში. _გრძელვადიანი მეხსიერებისთვის_ კი - მოძრავი საშუალოები. ასევე, Date-ს მიხედვით გამოვყავით წლის, თვის, კვირისა და დღის აღმნიშვნელი სვეტებიც.  isHoliday-ს დავუმატეთ 4 დღესასწაულის აღმნიშვნელი სვეტი (მადლიერების დღე, შობა, super bowl, labor day) და Days_To_Next_Holiday, Days_Since_Last_Holiday.   

LightGBM _მგრძნობიარეა ჰიპერპარამეტრების მიმართ_ და საჭიროებს რეგულირებას.  
დავატუნინგეთ შემდეგი პარამეტრები : learning rate, number of leaves, subsample, colsample_bytree. პლოტებზე ჩანს დარანკული ფიჩერები და თითოეული პარამეტრის მნიშვნელობა პროგნოზირებისთვის. საბოლოო შედეგი wmae = 2989.  

--------------------------------------------------------------------------------------------------------------

***Classical Statistical Time-Series Models***

1. **ARIMA** - დროითი მწკრივების ანალიზის მარტივი კლასიკური სტატისტიკური მოდელია.  
* AutoRegressive მოდელი მიმდინარე მნიშვნელობას წინა მნიშვნელობების წრფივი კომბინაციით იცნობს.   
* Integrated დროის სერიას სტაბილურ საშუალოსა და ვარიაციას ანიჭებს.   
* Moving Average წარსული შეცდომების მიხედვით ცვლის დღევანდელ მნიშვნელობას : თუ წინა weekly_sales ძალიან დაბალი მოუვიდა, დღევანდელ მნიშვნელობას გაზრდის.
  
მოდელი კარგად მუშაობს, თუ დროითი სერია სტაბილურია. SARIMA-გან განსხვავებით, _სეზონურობას ვერ აღიქვამს_. არის წრფივი და ვერ აფიქსირებს რთულ, არაწრფივ ნიმუშებს. არ ითვალისწინებს დამატებით მახასიათებლებს. შესაბამისად, ეს არქიტექტურა არაა იდეალური ამ ამოცანისთვის. საბოლოო wmae = 3762, რაც მოსალოდნელიც იყო.  


--------------------------------------------------------------------------------------------------------------

2. **SARIMA (Seasonal AutoRegressive Integrated Moving Average)** წარმოადგენს კლასიკურ სტატისტიკურ მოდელს, რომელიც გამოიყენება დროითი   მწკრივების პროგნოზირებისთვის, განსაკუთრებით მაშინ, როცა მონაცემებს აქვთ სეზონური და ტენდენციური კომპონენტები. მოდელი აერთიანებს კომპონენტებს: AR (AutoRegressive) — წარსული მნიშვნელობების გავლენა მიმდინარე მნიშვნელობაზე, MA (Moving Average) — წარსული შეცდომების გავლენა მიმდინარე მნიშვნელობაზე და ამ ყველაფრის სეზონური ვერსიები.

გამოვიყენე პარამეტრები:  
SARIMA(1,1,1)(1,1,1,52) = SARIMA(p,d,q)(P,D,Q,s)

მოდელი: გამოვიყენე SARIMA(1,1,1)(1,1,1,52) - სადაც autoregressive order არის 1, differencing - 1, moving average — 1, ხოლო სეზონური პარამეტრები მითითებულია 52 კვირაზე. თითოეული Store-Dept კომბინაციისთვის ცალკე ვწვრთნიდი მოდელს.   
შედეგი: WMAE = 1192.24, MAE = 1110.76. 200 კომბინაციიდან 189 წარმატებით დასრულდა (94.5% success rate).  

--------------------------------------------------------------------------------------------------------------

3. **SARIMAX** (SARIMA with eXogenous variables) - იგივე SARIMA-ს განახლებული ვერსიაა, რომელსაც დამატებით შეუძლია გარე ფაქტორების (exogenous variables) ჩართვა პროგნოზში.
   
მაგ.: თუ გაყიდვებზე გავლენას ახდენს CPI, unemployment, markdown ან დღესასწაულები — SARIMAX საშუალებას გაძლევს ეს ინფორმაცია პირდაპირ ჩართო მოდელში. 

ძირითადი უპირატესობა: შეუძლია დროითი მწკრივის გარდა, სხვა ცვლადების ეფექტის გათვალისწინება, განსაკუთრებით ძლიერია, როცა სხვა ცვლილებები მკვეთრად მოქმედებენ დროით სერიაზე.  
Sarima - დან გადავედით SARIMAX მოდელზე, სადაც ჩართული იყო 12 დროითი ფიჩერი (მაგ. დღესასწაულები, ტრენდის ინდიკატორები, სეზონური ცვლადები და სხვ.)  
Validation-სთვის გამოვიყენე 8-კვირიანი holdout და 50 შემთხვევითი Store-Dept კომბინაცია 3331-დან.  
ყველა 50 მოდელი წარმატებით დასრულდა (100%).  

შედეგები:  
მეტრიკა	მნიშვნელობა  
WMAE  -  1218.20
MAE	- 1207.74  
RMSE	- 2678.62   
R² - 	0.9883  
Holiday MAE	- 1240.06  
Regular MAE	- 1203.35  

--------------------------------------------------------------------------------------------------------------

4. **Prophet** - კლასიკური დროითი მწკრივების მოდელია. არის ინტუიციური და მარტივად გამოსაყენებელი. პროგნოზს აკეთებს სამი ძირითადი კომპონენტის გამოყენებით:   
* ტრენდი - აღნიშნავს ზოგადად მონაცემების ცვლილებას : აღმავალი, დაღმავალი თუ სტაბილური.  
* სეზონურობა - განმეორებითი პატერნები, იქნება ეს ყოველკვირეული, ყოველწლიური თუ სხვ.   
* დღესასწაულები(ნებაყოფლობითი) - კონკრეტული დღეების გავლენა მონაცემებზე.
    
ზოგადი ფორმულა შეგვიძლია ასე წარმოვადგინოთ : **y(t) = trend(t) + seasonality(t) + holiday_effects(t) + error**.  

იყენებს ფურიეს სერიებს სეზონურობის მოდელირებისთვის. მოდელი ავტომატურად ამუშავებს NaN მნიშვნელობებს. მაგრამ არ არის იდეალური რთული ან ზედმეტად არაწრფივი მონაცემებისთვის. შედეგის გასაუმჯობესელბად მივუთითე 4-ვე დღესასწაული. რეალურად, რადგან პიკები ყოველ დეკემბერს ხდება Prophet შეეცდება დეკემბერში გლუვი სეზონური მატების დაფიქსირებას. თუმცა, ვერ დააფიქსირებს დიდი გაყიდვების ზუსტ ზომას ან დროს. ეს პიკი ზედმეტად მკვეთრია იმისთვის, რომ წლიური სეზონურობის გლუვი მრუდით აისახოს და შეიძლება მოდელმა გამონაკლისადაც კი მიიჩნიოს - რაც ნიშნავს, რომ ტრენინგის დროს უგულებელყოფს მას. კეგლზე საბოლოოდ wmae მომცა 3216.   

--------------------------------------------------------------------------------------------------------------


***Deep Learning მოდელები***

1. **N-BEATS** (Neural Basis Expansion Analysis for Time Series) არის Deep Learning მოდელი, რომელიც სპეციალურად ერთცვლადიანი დროითი სერიების პროგნოზირებისთვისაა შექმნილი - მომავალი მნიშვნელობების პროგნოზირება მხოლოდ სამიზნე ცვლადის წარსულ დაკვირვებებზე დაყრდნობით ხდება. თავად სწავლობს მონაცემების სეზონურობას, ტენდენციასა და სტრუქტურას. არ სჭირდება მონაცემთა ინჟინერია, მთავარია დრო და სამიზნე გადაეცემოდეს. შეუძლია არაწრფივი და კომპლექსური მონაცემების დამუშავება. იყენებს ბლოკებს. თითოეული ბლოკი არა მხოლოდ მომავალს პროგნოზირებს (forecast), არამედ ცდილობს წარსულის რეკონსტრუქციას (backcast). მოდელი თუ სწორად ახსნის შემავალ მონაცემებს (მაგ., ტენდენცია თუ სეზონურობა), პროგნოზსაც ზუსტად გააკეთებს. ის, რისი ახსნაც საწყისმა ბლოკმა ვერ მოახერხა, გადაეცემა მომდევნოს.

საბოლოო პასუხი ყველა ბლოკის პროგნოზის ჯამია :
   **[Past Values] → [Block 1] → [Block 2] → ... → [Final Forecast]**

გვაქვს 3 ტიპის ბლოკი: _იდენტობა, ტრენდი და სეზონურობა_. ერთი ტიპის ბლოკები ქმნიან სტეკს.    

ზოგადად, ნეირონული ქსელები მგრძნობიარეა ინფუთის მასშტაბის მიმართ. რადგან თითოეულ მაღაზიასა და დეპარტამენტს საგრძნობლად განსხვავებული გაყიდვები აქვს, _რეკომენდებულია ნორმალიზაცია_. გამოვიყენე NeuralForecast ბიბლიოთეკა. შევქმენი ქვეკლასი N_beats, დავარეგულირე gamma, input_size (რამდენი კვირის ინფორმაცია გადაეცემა მოდელს წინასწარმეტყველებისთვის), horizon (რამდენი კვირა უნდა იწინასწარმეტყველოს), stack types, number of blocks, batch size. საბოლოო wmae მივიღე 2087, რაც არაა ცუდი შედეგი. 

--------------------------------------------------------------------------------------------------------------

2. **PatchTST** (Patching Time Series Transformer) არის თანამედროვე Deep Learning მოდელი, რომელიც გამოიყენება დროითი მწკრივების პროგნოზირებისთვის და ეფუძნება Transformer არქიტექტურას. მისი ძირითადი პრინციპი მდგომარეობს დროითი მწკრივის პატარა თანმიმდევრობებად — „პატჩებად“ დაყოფაში. თითოეული პატჩი ინდივიდუალურად გადაეცემა Transformer ბლოკებს, სადაც Multi-Head Self-Attention მექანიზმის მეშვეობით ხდება სექვენსის სხვადასხვა მონაკვეთს შორის დამოკიდებულებების დინამიური შეფასება. ეს მიდგომა მოდელს საშუალებას აძლევს ერთდროულად დააფიქსიროს როგორც ლოკალური (პატჩში არსებული მოკლევადიანი კავშირები), ასევე გლობალური (სექვენსის მთელ მანძილზე გამოკვეთილი ტენდენციები) დამოკიდებულებები. PatchTST გამოირჩევა ეფექტური გათვლითი სტრუქტურით, ვინაიდან პატჩებზე დაყოფა ამცირებს საერთო გამოთვლით სირთულეს და ამავე დროს ინარჩუნებს ყურადღების მექანიზმის ძლიერ მხარეს — დროითი ურთიერთკავშირების აღმოჩენას.
   
მოდელის კონფიგურაცია განისაზღვრა შემდეგნაირად:  
input_dim = 30 (შერჩეული ფიჩერების რაოდენობა)  
patch_size = 12 (თითოეული პატჩის სიგრძე კვირებში)  
d_model = 64 (პატჩის embedding-ის ზომა)  
n_heads = 8 (Self-Attention თავების რაოდენობა)  
n_layers = 4 (Transformer ფენების რაოდენობა)  
dropout = 0.15  
batch_size = 64  

მოდელის შედეგმა აჩვენა, რომ WMAE დაფიქსირდა 3232.47, რაც მიუთითებს იმაზე, რომ მოდელი აფიქსირებდა დროით შაბლონებს და სეზონურ ცვლილებებს, თუმცა მისი სიზუსტე საშუალო იყო სხვა მოდელებთან შედარებით.  
სწავლების პროცესში ტესტირება ჩავატარე რამდენიმე ალტერნატიული კონფიგურაციით, მათ შორის:  
n_heads = 6 და 12  
d_model = 32 და 128  
dropout = 0.1 და 0.25 

ასევე ვცადე patch_size-ის ცვლილება 8 და 16 მნიშვნელობებზე. აღნიშნული ვარიანტების უმეტესობაში შედეგები ან გაუარესდა, ან ზუსტად ისეთივე იყო, რაც მიუთითებს, რომ ამ კონკრეტული სეტინგით მოდელმა მიაღწია ოპტიმალურ ბალანსს გამოთვლით ეფექტურობასა და სიზუსტეს შორის.

--------------------------------------------------------------------------------------------------------------

3. **DLinear (Decomposition Linear)** არის ეფექტური დროის სერიების პროგნოზირებისთვის გათვლილი მოდელი, რომელიც დაფუძნებულია სიგნალის დეკომპოზიციაზე. მონაცემები იყოფა ორ კომპონენტად: ტრენდად და სეზონურობად. ტრენდი არის გრძელვადიანი ტენდენცია, რომელსაც მოდელი გამოითვლის საშუალო მოძრავი საშუალოს (AvgPool1d) გამოყენებით, წინწასწარ განსაზღვრული, გადაცემული პერიოდისთვის.
     
სეზონურობა კი გამოყოფილია თავდაპირველი სიგნალისგან ტრენდის გამოკლებით და აჩვენებს მონაცემების პერიოდულ ცვლილებებს.  
მოდელი თითოეული ფიჩერისთვის ცალ-ცალკე გამოთვლის ტრენდს და სეზონურობას და თითოეულ კომპონენტზე მიმართავს საკუთარ linear ფენას (მარტივი ხაზოვანი მოდელი), შემდეგ კი მათი შედეგებს აჯამებს საბოლოო პროგნოზის მისაღებად.  

DLinear მოდელი მუშაობდა 12 კვირიანი ისტორიის საფუძველზე. პრეპროცესინგის ეტაპზე, დავამატეთ ზემოთ განხილული ცვლადები, რომლებიც მოიცავდა დროის მახასიათებლებს, დღესასწაულებს, lag მნიშვნელობებს, მაღაზია-დეპარტამენტის სტატისტიკას, ფასდაკლებებს და ეკონომიკურ ინდიკატორებს. ფიჩერების არჩევისთვის გამოვიყენე SelectKBest, რის შედეგადაც ავარჩიე საუკეთესო 30 ფიჩერი. მონაცემების ნორმალიზაციისთვის გამოვიყენე StandardScaler. შედეგების მიხედვით, DLinear მოდელმა აჩვენა ვალიდაციის შედეგი WMAE = 2528.44 და R² = 0.92. მოდელი გავწვრთენი Early Stopping-ით 100 ეპოქამდე, ხოლო სასწავლო პროცესში გამოვიყენე Adam optimizer learning rate scheduling-ით, სადაც საწყისი learning rate იყო 0.003 და ეტაპობრივად მცირდებოდა 0.00005-მდე.

პარამეტრები:  
Seq-length (წინასწარი ისტორიის ზომა) = 12  
Pred-length (პროგნოზის სიგრძე) = 1  
Learning rate = 0.003 (კლება 0.00005-მდე)  
optimizer	- Adam ოპტიმიზატორი  
scheduler step size	- 15 (learning rate-ის ცვლილების ინტერვალი)  
scheduler gamma -	0.5 (learning rate-ის შემცირების ფაქტორი)  
early stopping	patience = 12 (სწავლების შეჩერება, თუ 12 ეპოქაში გაუმჯობესება არ არის)  

--------------------------------------------------------------------------------------------------------------



