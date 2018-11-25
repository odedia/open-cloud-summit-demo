# open-cloud-summit-demo

Open Cloud Summit
-----------------

# Part 1 - Generate beer
------------------------------

start.spring.io

project name: beer-service

```
Web
Actuator
HATEOAS
Rest Repositories
JPA
H2
MySQL
Lombok
Redis
```

ggreeting
---------
```
@RestController
@Slf4j
class BeerGreeting {

	@Autowired
	Environment env;
	
	@GetMapping("/greeting")
	String greeting(){
		log.info("Returning greeting from service...");
		return "Greetings from Open Cloud Summit! I am instance #" + env.getProperty("CF_INSTANCE_INDEX");
	}
}
```

# Part 2 - cf push
------------------------------

mvn clean package -DskipTests
mvn spring-boot:run

cf push -p target/beer-service-0.0.1-SNAPSHOT.jar beer-service

Part 3 - Apps Manager
------------------------------

Show actuator events, (down - no redis), logs, trace, PCF metrics, scale the app to 2.

Part 4 - MySQL
------------------------------
Create service from UI. Can also do:

cf create-service p.mysql db-small demo-mysql
cf create-service p.redis cache-small redis

application.properties: 

spring.jpa.generate-ddl=true
----------

bbeer
------
```
@Entity
@Data
@NoArgsConstructor
@AllArgsConstructor
class Beer {
	@Id
	@GeneratedValue
	private Long id;
	private String name;
	private Double price;
}
```

rrepo
------
```
@RepositoryRestResource(path = "beers")
interface BeerRepository extends JpaRepository<Beer, Long> {

}
```
rrunner
-------

In BeerApplication:

```	@Autowired
	private RedisTemplate<String, String> template;


	@Bean
	CommandLineRunner runner(BeerRepository br){
		return args -> {
			if (br.count() > 0) return;

			Arrays.asList("")
			
			.forEach(x -> br.save(new Beer(null, x, 10.0)));
			
			br.findAll().forEach(System.out::println);
		};
    		template.opsForValue().set("count", String.valueOf(br.count()));
	}
```

Replace greeting:

greett
------

```
@RestController
@Slf4j
class BeerGreeting {
	@Autowired
	private RedisTemplate<String, Integer> template;

	@Autowired
	Environment env;

	@Autowired
	BeerRepository br;

	@GetMapping("/greeting")
	String greeting() {
		Random rand = new Random();

		Integer count = template.opsForValue().get("count");

		Long beerIndex = Long.valueOf(rand.nextInt(count) + 1);

		log.info("Returning greeting from service...");
		
		simulateLoad();
		
		return "Greetings from Open Cloud Summit! I am instance "  
				+ env.getProperty("CF_INSTANCE_INDEX") + "\nHave a beer: "
				+ br.findById(beerIndex).get().getName();
	}
	
	private void simulateLoad() {
		// Creation
		FakeLoad fakeload = FakeLoads.create().lasting(2, TimeUnit.SECONDS).withCpu(50);

		// Execution
		FakeLoadExecutor executor = FakeLoadExecutors.newDefaultExecutor();
		executor.execute(fakeload);
	}

}
```
Add to pom.xml:

lload
------

Add to pom.xml:

		<dependency>
			<groupId>com.martensigwart</groupId>
			<artifactId>fakeload</artifactId>
			<version>0.4.0</version>
		</dependency>





Create manifest.yml:
```
---
applications:
- name: beer-service
  path: target/beer-service-0.0.1-SNAPSHOT.jar
  services:
  - mysql
  - redis
```

mvn clean package -DskipTests && cf push

Show random beer being handed



Part 6 - Autoscaler
-------------------
Define autoscaler: CPU min 20% max 50%, Latency 50-150ms, 2-10 instances

Ask audience to login to http://bit.ly/beer1234

Change return string to 

*** BRAND NEW GREETING *** from Open Cloud Summit!

mvn clean package -DskipTests && cf bgd beer-service -f manifest.yml --delete-old-apps


for ((i=1;i<=100;i++)); do curl https://beer-service.apps.pcfone.io/greeting; done


Part 7 - Others
--------------------
cf ssh dotnet (twice...)

https://axontrader.cfapps.io/

for ((i=1;i<=10;i++)); do curl https://beer-service.apps.pcfone.io/greeting; done
