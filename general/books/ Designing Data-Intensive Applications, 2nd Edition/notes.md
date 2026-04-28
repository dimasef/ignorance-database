# The Manager's Path

## How to Use These Notes

- `TL;DR` — a 2-4 sentence summary of the chapter's core idea.
- `Key Ideas` — short, high-signal takeaways without extra detail.
- `My Notes` — personal observations, associations, disagreements, and examples from experience.
- `Actions` — things I want to try in practice.
- `Questions` — things that are still unclear or worth revisiting later.

## Inbox

- If you are interested in improving on purely the people management side of leadership, books like _First, Break All the Rules_ are excellent references.

---

## Chapter 1. Trade-Offs in Data Systems Architecture

**EN:**  
Data-intensive applications face challenges in storing, processing, and managing large volumes of data across multiple systems. Modern applications need to combine different tools—databases, caches, search indexes, stream processing, and batch processing—while understanding trade-offs between different technologies. The key is learning to ask the right questions to evaluate which approach best serves your specific needs, recognizing that no single solution works for all scenarios.

**UA:**  
Додатки, інтенсивні за даними, стикаються з викликами зберігання, обробки та управління великими обсягами даних у кількох системах. Сучасні застосунки повинні комбінувати різні інструменти—бази даних, кеш, пошукові індекси, потокову обробку та пакетну обробку—зважаючи на компроміси між різними технологіями. Ключ до успіху—вчитися задавати правильні запитання для оцінки систем, розуміючи, що немає універсального рішення для всіх сценаріїв.

### 1.1 Operational Versus Analytical Systems

**EN:**  
Organizations typically have two distinct types of data systems: operational systems where backend services create and modify data based on user actions, and analytical systems that contain read-only copies optimized for business intelligence and data science work. While backend engineers manage operational systems, business analysts and data scientists work with analytical systems to generate insights and predictions. This separation exists for good reasons, and specialized roles like data engineers and analytics engineers have emerged to bridge the gap and manage the overall data infrastructure.

**UA:**  
Організації зазвичай мають два окремі типи систем даних: операційні системи, де backend-сервіси створюють та змінюють дані на основі дій користувачів, та аналітичні системи з копіями даних, оптимізованими для бізнес-аналітики та data science. Хоча backend-інженери керують операційними системами, бізнес-аналітики та дейта-сайєнтисти працюють з аналітичними системами для отримання інсайтів і прогнозів. Цей поділ існує з вагомих причин, і виникли спеціалізовані ролі—дейта-інженери та аналітики-інженери—для координації та управління загальною інфраструктурою даних.

#### 1.1.1 Characterizing Transaction Processing and Analytics

**EN:**  
OLTP (Online Transaction Processing) and OLAP (Online Analytical Processing) represent two fundamentally different database access patterns. OLTP systems handle frequent small queries that fetch or modify individual records—typical of user-facing applications—while OLAP systems scan massive datasets to compute aggregates for business intelligence and decision-making. OLTP systems run predefined queries for security and performance reasons, whereas OLAP systems allow ad-hoc exploration. A newer category, real-time analytics systems like Pinot and Druid, bridges the gap by ingesting data continuously and optimizing for low-latency analytical queries in user-facing products.

**UA:**  
OLTP (Online Transaction Processing) та OLAP (Online Analytical Processing) представляють два принципово різних паттерни доступу до баз даних. OLTP-системи обробляють часті невеликі запити, що отримують або змінюють окремі записи—типові для користувацьких додатків, тоді як OLAP-системи сканують величезні обсяги даних для обчислення агрегатів для бізнес-аналітики. OLTP-системи запускають заздалегідь визначені запити з міркувань безпеки та продуктивності, водночас OLAP-системи дозволяють довільні розвідувальні запити. Новіша категорія—системи real-time аналітики, як Pinot та Druid—заповнює розрив, обробляючи дані безперервно та оптимізуючи для низькольатентних аналітичних запитів у користувацьких продуктах.

#### 1.1.2 Data Warehousing

**EN:**  
Data warehousing emerged in the late 1980s as companies separated analytics from their OLTP systems to avoid performance impacts and data silos across multiple operational databases. A data warehouse is a centralized, read-only repository that stores transformed data from all operational systems via ETL (extract-transform-load) processes, allowing analysts to run complex queries without affecting production systems. While HTAP (hybrid transactional/analytical processing) systems attempt to combine both workloads, data warehouses remain essential because enterprises typically have hundreds of separate operational databases but need a single unified analytics database. The trend reflects a broader move toward specialized systems optimized for specific workloads as scale increases.

**UA:**  
Сховища даних виникли в кінці 1980-х, коли компанії відокремили аналітику від OLTP-систем, щоб уникнути впливу на продуктивність та проблем з силосами даних у кількох операційних базах. Сховище даних—це централізоване, лише для читання сховище, що зберігає перетворені дані з усіх операційних систем через ETL-процеси (витяг-трансформація-завантаження), дозволяючи аналітикам запускати складні запити без впливу на виробничі системи. Хоча HTAP-системи (гібридна обробка транзакцій/аналітики) намагаються поєднати обидва навантаження, сховища даних залишаються суттєвими, оскільки підприємства зазвичай мають сотні окремих операційних баз, але потребують єдиної аналітичної бази. Цей тренд відображає ширший рух до спеціалізованих систем, оптимізованих для конкретних навантажень при зростанні масштабу.

#### 1.1.3 From data warehouse to data lake

**EN:**  
Data lakes emerged to serve data scientists who need flexibility beyond what traditional SQL-based data warehouses offer. While data warehouses use relational schemas optimized for business analysts' SQL queries, data scientists often require custom code for feature engineering and ML tasks that don't fit well into SQL. A data lake is a centralized repository that stores raw data in any format—database records, text, images, videos, sensor data—without imposing a fixed schema or data model. Data lakes are more flexible and cost-effective than relational storage, using commoditized object stores. Modern data pipelines often flow from operational systems to a data lake (raw data) and then to a data warehouse (structured data), allowing each consumer to transform data according to their specific needs—the "sushi principle" of keeping raw data untouched.

**UA:**  
Data lakes виникли для обслуговування дейта-сайєнтистів, яким потрібна гнучкість за межами традиційних SQL-орієнтованих сховищ даних. Хоча сховища даних використовують реляційні схеми, оптимізовані для SQL-запитів бізнес-аналітиків, дейта-сайєнтисти часто потребують користувацького коду для feature engineering та задач ML, які не вписуються добре в SQL. Data lake—це централізоване сховище, що зберігає сирові дані в будь-якому форматі—записи баз даних, текст, зображення, відео, дані сенсорів—без нав'язування фіксованої схеми або моделі даних. Data lakes гнучкіші та економічніші за реляційне сховище, використовуючи стандартизовані об'єктні сховища. Сучасні конвеєри даних часто текуть від операційних систем до data lake (сирові дані) і далі до сховища даних (структуровані дані), дозволяючи кожному споживачу трансформувати дані відповідно до своїх потреб—принцип «sushi» збереження сирих, непокидених даних.

#### 1.1.4 Systems of Record and Derived Data

**EN:**  
Systems of record (source of truth) hold the authoritative version of data where facts are written first and stored exactly once, while derived data systems contain transformed or processed copies of that data—caches, indexes, materialized views, ML models. Derived data is technically redundant but essential for read performance and enabling different views of the same data. Analytical systems are typically derived systems, while operational systems contain a mix of both. The distinction is not about the tool used but how it's applied—clarifying which data is derived from which source helps bring order to complex architectures. Keeping derived data in sync with the system of record requires dedicated data pipeline processes, especially when multiple systems need to be composed together.

**UA:**  
Systems of record (джерело правди) зберігають авторитетну версію даних, де факти записуються першими та зберігаються рівно один раз, тоді як derived data systems містять трансформовані або оброблені копії цих даних—кеші, індекси, materialized views, ML-моделі. Derived data технічно надлишкова, але необхідна для продуктивності читання та забезпечення різних поглядів на ті самі дані. Аналітичні системи зазвичай є derived systems, тоді як операційні системи містять суміш обох. Різниця полягає не в інструменті, а в тому, як він застосовується—розуміння того, які дані є похідними від яких джерел, допомагає впорядкувати складні архітектури. Підтримка derived data в синхронізації з system of record вимагає спеціальних процесів data pipeline, особливо коли кілька систем потрібно компонувати разом.

### 1.2 Cloud Versus Self-Hosting

**EN:**  
Organizations face a fundamental build-vs-buy decision for software: whether to develop in-house or outsource. The guiding principle is that core competencies and competitive advantages should stay in-house, while routine, non-core tasks should be delegated to vendors. The spectrum ranges from fully custom in-house software to cloud SaaS products, with a middle ground of self-hosted open-source or commercial software deployed on your own hardware or cloud VMs (IaaS). The choice depends on business priorities rather than technical factors alone, and deployment tooling like Kubernetes, while relevant, has less architectural impact on data systems than the fundamental hosting decision itself.

**UA:**  
Організації стикаються з фундаментальним рішенням «будувати чи купувати» для програмного забезпечення: розробляти власними силами чи аутсорсити. Керівний принцип полягає в тому, що ключові компетенції та конкурентні переваги мають залишатися всередині компанії, тоді як рутинні, неосновні задачі варто делегувати вендорам. Спектр варіантів варіюється від повністю кастомного in-house програмного забезпечення до хмарних SaaS-продуктів, з середньою ланкою у вигляді self-hosted open-source або комерційного ПЗ, розгорнутого на власному залізі або хмарних VM (IaaS). Вибір залежить від бізнес-пріоритетів, а не лише від технічних факторів, а інструменти розгортання, як Kubernetes, хоча і важливі, мають менший архітектурний вплив на системи даних, ніж саме фундаментальне рішення щодо хостингу.

#### 1.2.1 Pros and Cons of Cloud Services

**EN:**  
Cloud services offer significant advantages—faster setup, elastic scaling for variable workloads, and outsourced operational expertise—but come with serious trade-offs. The cost benefit over self-hosting depends heavily on your team's skills and workload predictability; for stable, predictable loads, owning hardware is often cheaper. The biggest downsides are loss of control: limited customization, dependency on vendor uptime and roadmap, vendor lock-in due to non-standard APIs, geopolitical risks, and security/compliance complications. Despite these risks, cloud adoption continues to grow, often through hybrid approaches, though latency-sensitive or highly specialized systems still require in-house infrastructure.

**UA:**  
Cloud-сервіси пропонують суттєві переваги—швидке розгортання, еластичне масштабування для змінних навантажень та аутсорс операційної експертизи—але мають серйозні компроміси. Економічна вигода порівняно з self-hosting значно залежить від навичок команди та передбачуваності навантаження; для стабільних, передбачуваних навантажень власне залізо часто дешевше. Найбільші недоліки—втрата контролю: обмежена кастомізація, залежність від аптайму та roadmap вендора, vendor lock-in через нестандартні API, геополітичні ризики та ускладнення щодо безпеки і відповідності нормам. Незважаючи на ці ризики, хмарна міграція продовжує зростати, часто через гібридні підходи, хоча latency-чутливі або вузькоспеціалізовані системи все ще потребують власної інфраструктури.

#### 1.2.2 Cloud Native System Architecture

**EN:**  
Cloud native architecture goes beyond simply hosting existing software in the cloud—it means designing systems from the ground up to leverage cloud services, achieving better performance, faster failure recovery, elastic scaling, and support for larger datasets. The key distinction from traditional IaaS (running software on VMs) is that cloud native systems build upon lower-level cloud services like object storage (S3, Azure Blob Storage) as foundational layers, creating higher-level abstractions on top. For example, Snowflake builds its analytical database on S3, while other services build on Snowflake in turn. Higher-level abstractions suit well-defined use cases with less operational overhead, while lower-level components remain necessary when no existing high-level solution fits your needs.

**UA:**  
Cloud native архітектура виходить за межі простого хостингу існуючого програмного забезпечення в хмарі—вона означає проектування систем з нуля для використання хмарних сервісів, досягаючи кращої продуктивності, швидшого відновлення після збоїв, еластичного масштабування та підтримки більших датасетів. Ключова відмінність від традиційного IaaS (запуск ПЗ на VM) полягає в тому, що cloud native системи будуються на основі низькорівневих хмарних сервісів, таких як object storage (S3, Azure Blob Storage), як фундаментальних шарів, створюючи на їх основі абстракції вищого рівня. Наприклад, Snowflake будує свою аналітичну базу даних на S3, тоді як інші сервіси будуються на Snowflake. Абстракції вищого рівня підходять для чітко визначених use cases з меншими операційними витратами, тоді як низькорівневі компоненти залишаються необхідними, коли жодне існуюче high-level рішення не відповідає вашим потребам.

#### 1.2.2.1 Separation of storage and compute

**EN:**  
Cloud native systems fundamentally separate storage from compute, unlike traditional architectures where both reside on the same machine. Instead of relying on local or virtual disks (which introduce network overhead and availability issues), cloud native systems use dedicated storage services like S3 for large data blocks while managing smaller values separately. This disaggregation means computation and storage can scale independently—data lives in object storage while processing runs elsewhere, transferring data over the network as needed. Cloud native systems are also typically multitenant, sharing hardware across many customers for better utilization and scalability, but requiring careful engineering to maintain performance isolation and security between tenants.

**UA:**  
Cloud native системи фундаментально розділяють сховище та обчислення, на відміну від традиційних архітектур, де обидва знаходяться на одній машині. Замість покладання на локальні або віртуальні диски (які вносять мережеві затримки та проблеми доступності), cloud native системи використовують виділені сервіси сховища, як S3, для великих блоків даних, управляючи меншими значеннями окремо. Це розмежування означає, що обчислення та сховище можуть масштабуватися незалежно—дані зберігаються в object storage, тоді як обробка виконується в іншому місці, передаючи дані через мережу за потреби. Cloud native системи також зазвичай є multitenant, розділяючи залізо між багатьма клієнтами для кращого використання ресурсів та масштабованості, але вимагаючи ретельної інженерії для підтримки ізоляції продуктивності та безпеки між клієнтами.

#### 1.2.3 Operations in the Cloud Era

**EN:**  
Cloud services have transformed the operations role from managing individual machines (capacity planning, provisioning, patching) toward managing services and their integrations. The DevOps/SRE philosophy emphasizes automation, ephemeral infrastructure, and frequent deployments over manual one-off tasks. While cloud providers handle low-level infrastructure reliability, their customers still need operations—focused on choosing the right services, integrating them, and optimizing costs (since capacity planning effectively becomes financial planning). Challenges like security, service monitoring, and cross-vendor integration remain the customer's responsibility, and the lack of standards for service integration requires significant manual effort. The cloud changes the nature of operations but doesn't eliminate the need for it.

**UA:**  
Cloud-сервіси трансформували роль операцій від управління окремими машинами (планування ємності, provisioning, патчинг) до управління сервісами та їх інтеграціями. Філософія DevOps/SRE наголошує на автоматизації, ephemeral інфраструктурі та частих деплоях замість ручних разових задач. Хоча хмарні провайдери займаються надійністю низькорівневої інфраструктури, їхні клієнти все одно потребують операцій—зосереджених на виборі правильних сервісів, їх інтеграції та оптимізації витрат (оскільки планування ємності фактично стає фінансовим плануванням). Такі виклики, як безпека, моніторинг сервісів та міжвендорна інтеграція залишаються відповідальністю клієнта, а відсутність стандартів для інтеграції сервісів вимагає значних ручних зусиль. Cloud змінює природу операцій, але не усуває потребу в них.

### Key Ideas

- A good manager neither disappears nor suffocates the team with control.
- One-on-ones matter because they create trust and space for honest discussion, not because they track status.
- As you become more senior, feedback gets less frequent and more strategic, so you need to ask for it directly.
- Career growth cannot be fully delegated to your manager; you have to drive it yourself.
- Being well-managed is also your responsibility: clarify what you want, speak up, and make the relationship productive.
- Choosing a manager is often as important as choosing the role itself.

### My Notes

- The manager here is framed more as a force multiplier than as a controller.
- A useful takeaway is that management is a two-way relationship: the employee also has to bring clarity, initiative, and realistic expectations.

### Actions

- Review which one-on-one questions actually help and which ones only fill calendar space.
- Write down what kind of feedback I need from my manager right now.
- Define what I currently want from my role and what support would actually move me forward.
- Pay more attention to manager quality when evaluating future opportunities.

### Questions

- Where is the line between useful managerial attention and micromanagement?
- How often do one-on-ones stay valuable at the senior or lead level?
- What is the best way to ask for more strategic feedback without sounding vague?
- How do you evaluate a manager well before joining a team?

---
