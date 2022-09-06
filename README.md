# CreateRESTAPIsWithJavaAndSpringBoot

Spring Boot minimum konfigürasyonla REST API'ler oluşturmanıza olanak tanır. XML yapılandırmalarına ihtiyaç duymaz. 

#### Spring Initializr;
- Group : Projeyi oluşturan grubu veya kuruluşu belirten temel paket adıdır.
- Artifact : Proje adıdır. 
- Name : Spring Boot'un proje için giriş noktası oluştururken kullanacağı uygulamanızın görünen adıdır.
- Description : Proje hakkında açıklama

#### Dependencies;
- Spring Web: RESTful web uygulamaları geliştirmek için gereklidir.
- Spring Data JPA: Database verilerine erişmek için geçerlidir. CRUD işlemleri için yardımcı yöntemler sağlayan JPA üzerinden bir soyutlamadır. JDBC ile yaptığımız gibi sorgu yazma ihtiyacını ortadan kaldırır.
- MySQL Driver: MySQL veritabanına bağlanmak için gereklidir.


@SpringBootApplication;
- @EnableAutoConfiguration : Spring boot otomatik yapılandırma özelliğini etkinleştirir.
- @Configuration : Özel bir yapılandırma sınıfı oluşturur.
- @ComponentScan : Spring Boot'un paketi Service, Controller, Repository vb. bileşenler için bean oluşturur.

#### Sub-packages;
- DAO : The DAO (data access layer) veritabanına bağlanmak ve veritabanında depolanan verilere erişmek için bir interface sağlar.
- Repository : Veritabanına bağlanan ve verilere erişen DAO katmanına benzer. Ancak repository, DAO katmanına kıyasla daha büyük bir soyutlama sağlar. Her sınıf, bir varlığa erişmek ve bunları değiştirmekten sorumludur.
- Service : Verileri almak ve üzerinde iş mantığı gerçekleştirmek için DAO katmanını çağırır. Service katmanı iş mantığı, alınan veriler üzerinde hesaplamalar yapmak, verileri bir mantığa göre filtrelemek vb. olabilir.
- Model : Java nesnelerini içerir.
- Controller : Bir REST API için bir istek geldiğinde çağrılan en üst katmandır. Controller, REST API isteğini işleyecek, bir veya daha fazla hizmeti çağıracak ve istemciye bir HTTP yanıtı döndürecektir.

#### Model class'ı;
Employee

```
package com.example.employee.model;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.Table;

@Entity
@Table(name = "employee")
public class Employee {
        
        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        @Column(name="emp_id")
            private Long id;
        
        @Column(name="first_name")
        private String firstName;
        
        @Column(name="last_name")
        private String lastName;
        
        @Column(name="email_id")
        private String emailId;
}
```

- @Entity : Classın veritabanı tablosuna eşlendiğini belirtir.
- @Table : Hangi tablo ile eşleşeceğini belirtir.
- @Column : Variable eşleşeceği veritabanı tablosundaki alanı belirtir.
- @Id : Primary key
- @GeneratedValue : Primary key oluşturmak için kullanılan stratejiyi belirtir.

- GenerationType.AUTO : Bu, Spring Boot tarafından kullanılan default stratejidir. 
- GenerationType.IDENTITY : Primary key stratejisini belirlemek için veritabanı id sütununu kullanır. Örneğin, emp_id çalışan tablosunu oluştururken veritabanında sütunu otomatik artış olarak tanımladınız. Şimdi bu stratejiyi kullandığınızda, 1'den başlayarak ve tabloya her yeni satır eklendiğinde artan benzersiz bir primary key oluşturulur.
- GenerationType.SEQUENCE : Primary key oluşturmak için veritabanı sırasını kullanır.
- GenerationType.TABLE- Primary key oluşturmak için bir veritabanı tablosu kullanır.


#### Repository class;

```
package com.example.employee.repository;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

import com.example.employee.model.Employee;

@Repository
public interface EmployeeRepository extends JpaRepository<Employee, Long> {

}
```
#### Service class;
```
package com.example.employee.service;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import com.example.employee.model.Employee;
import com.example.employee.repository.EmployeeRepository;
import java.util.List;

@Service
public class EmployeeService {

        @Autowired
            EmployeeRepository empRepository;        
}
```

- @Autowired : Bir sınıf nesnesini soyutlaştırmak için kullanılır.

```
// CREATE 
public Employee createEmployee(Employee emp) {
    return empRepository.save(emp);
}

// READ
public List<Employee> getEmployees() {
    return empRepository.findAll();
}

// DELETE
public void deleteEmployee(Long empId) {
    empRepository.deleteById(empId);
}

// UPDATE
public Employee updateEmployee(Long empId, Employee employeeDetails) {
        Employee emp = empRepository.findById(empId).get();
        emp.setFirstName(employeeDetails.getFirstName());
        emp.setLastName(employeeDetails.getLastName());
        emp.setEmailId(employeeDetails.getEmailId());
        
        return empRepository.save(emp);                                
}
```

#### Contoller class;

```
package com.example.employee.controller;

import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

import com.example.employee.model.Employee;
import com.example.employee.service.EmployeeService;

@RestController
@RequestMapping("/api")
public class EmployeeController {
        @Autowired
        EmployeeService empService;

}
```
- @RequestMapping : Bu controllerde oluşturulan tüm REST API'ler için bir temel URL tanımlar.
- @RestController
- @Controller : Spring Boota bu classın bir controller olduğunu belirtir.
- @ResponseBody : Metod dönüş değerinin REST API için response body belirtir.

#### Controller classı için;
```
@RequestMapping(value="/employees", method=RequestMethod.POST)
public Employee createEmployee(@RequestBody Employee emp) {
    return empService.createEmployee(emp);
}

@RequestMapping(value="/employees", method=RequestMethod.GET)
public List<Employee> readEmployees() {
    return empService.getEmployees();
}

@RequestMapping(value="/employees/{empId}", method=RequestMethod.PUT)
public Employee readEmployees(@PathVariable(value = "empId") Long id, @RequestBody Employee empDetails) {
    return empService.updateEmployee(id, empDetails);
}

@RequestMapping(value="/employees/{empId}", method=RequestMethod.DELETE)
public void deleteEmployees(@PathVariable(value = "empId") Long id) {
    empService.deleteEmployee(id);
}
```




