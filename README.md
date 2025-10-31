Part (a): Dependency Injection in Spring Using Java-Based Configuration
code: 
package com.example.springdi;

public class Course {
    private String courseName;

    public Course(String courseName) {
        this.courseName = courseName;
    }

    public void showCourse() {
        System.out.println("Enrolled in course: " + courseName);
    }
}

package com.example.springdi;

public class Student {
    private Course course;

    // Constructor-based dependency injection
    public Student(Course course) {
        this.course = course;
    }

    public void displayInfo() {
        System.out.println("Student is studying:");
        course.showCourse();
    }
}

package com.example.springdi;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class AppConfig {

    @Bean
    public Course courseBean() {
        return new Course("Spring Framework Fundamentals");
    }

    @Bean
    public Student studentBean() {
        // Inject Course dependency into Student
        return new Student(courseBean());
    }
}

package com.example.springdi;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class MainApp {
    public static void main(String[] args) {
        // Initialize Spring context
        AnnotationConfigApplicationContext context =
                new AnnotationConfigApplicationContext(AppConfig.class);

        // Get bean and call method
        Student student = context.getBean(Student.class);
        student.displayInfo();

        context.close();
    }
}

Part (b): Hibernate Application for Student CRUD Operations
code:
package com.example.hibernatecrud;

import javax.persistence.*;

@Entity
@Table(name = "student")
public class Student {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;

    @Column(name = "name")
    private String name;

    @Column(name = "department")
    private String department;

    @Column(name = "marks")
    private double marks;

    public Student() {}

    public Student(String name, String department, double marks) {
        this.name = name;
        this.department = department;
        this.marks = marks;
    }

    // Getters and Setters
    public int getId() { return id; }
    public String getName() { return name; }
    public String getDepartment() { return department; }
    public double getMarks() { return marks; }

    public void setName(String name) { this.name = name; }
    public void setDepartment(String department) { this.department = department; }
    public void setMarks(double marks) { this.marks = marks; }

    @Override
    public String toString() {
        return id + " | " + name + " | " + department + " | " + marks;
    }
}

package com.example.hibernatecrud;

import org.hibernate.SessionFactory;
import org.hibernate.cfg.Configuration;

public class HibernateUtil {
    private static final SessionFactory sessionFactory;

    static {
        try {
            sessionFactory = new Configuration()
                    .configure("hibernate.cfg.xml")
                    .addAnnotatedClass(Student.class)
                    .buildSessionFactory();
        } catch (Throwable ex) {
            System.err.println("SessionFactory creation failed: " + ex);
            throw new ExceptionInInitializerError(ex);
        }
    }

    public static SessionFactory getSessionFactory() {
        return sessionFactory;
    }
}

package com.example.hibernatecrud;

import org.hibernate.*;
import java.util.List;

public class StudentDAO {
    public void saveStudent(Student s) {
        Session session = HibernateUtil.getSessionFactory().openSession();
        Transaction tx = session.beginTransaction();
        session.save(s);
        tx.commit();
        session.close();
        System.out.println("Student saved successfully!");
    }

    public List<Student> getAllStudents() {
        Session session = HibernateUtil.getSessionFactory().openSession();
        List<Student> list = session.createQuery("from Student", Student.class).list();
        session.close();
        return list;
    }

    public void updateStudent(int id, String name, double marks) {
        Session session = HibernateUtil.getSessionFactory().openSession();
        Transaction tx = session.beginTransaction();
        Student s = session.get(Student.class, id);
        if (s != null) {
            s.setName(name);
            s.setMarks(marks);
            session.update(s);
            tx.commit();
            System.out.println("Student updated!");
        } else {
            System.out.println("Student not found!");
        }
        session.close();
    }

    public void deleteStudent(int id) {
        Session session = HibernateUtil.getSessionFactory().openSession();
        Transaction tx = session.beginTransaction();
        Student s = session.get(Student.class, id);
        if (s != null) {
            session.delete(s);
            tx.commit();
            System.out.println("Student deleted!");
        } else {
            System.out.println("Student not found!");
        }
        session.close();
    }
}

package com.example.hibernatecrud;

import java.util.List;

public class MainApp {
    public static void main(String[] args) {
        StudentDAO dao = new StudentDAO();

        // Create
        dao.saveStudent(new Student("Alice", "CS", 85.5));

        // Read
        List<Student> students = dao.getAllStudents();
        students.forEach(System.out::println);

        // Update
        dao.updateStudent(1, "Alice Johnson", 90.0);

        // Delete
        dao.deleteStudent(1);

        HibernateUtil.getSessionFactory().close();
    }
}

Part (c): Transaction Management Using Spring and Hibernate
code: 
package com.example.bankapp;

import javax.persistence.*;

@Entity
@Table(name = "account")
public class Account {
    @Id
    private int accountId;

    @Column
    private String holderName;

    @Column
    private double balance;

    public Account() {}

    public Account(int accountId, String holderName, double balance) {
        this.accountId = accountId;
        this.holderName = holderName;
        this.balance = balance;
    }

    public int getAccountId() { return accountId; }
    public String getHolderName() { return holderName; }
    public double getBalance() { return balance; }

    public void setBalance(double balance) { this.balance = balance; }
}

package com.example.bankapp;

import org.hibernate.SessionFactory;
import org.hibernate.Session;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Repository;

@Repository
public class AccountDAO {

    @Autowired
    private SessionFactory sessionFactory;

    public Account getAccount(int id) {
        return sessionFactory.getCurrentSession().get(Account.class, id);
    }

    public void updateAccount(Account account) {
        sessionFactory.getCurrentSession().update(account);
    }
}

package com.example.bankapp;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
public class BankService {

    @Autowired
    private AccountDAO dao;

    @Transactional
    public void transferMoney(int fromId, int toId, double amount) {
        Account fromAcc = dao.getAccount(fromId);
        Account toAcc = dao.getAccount(toId);

        if (fromAcc.getBalance() < amount) {
            throw new RuntimeException("Insufficient funds in account " + fromId);
        }

        fromAcc.setBalance(fromAcc.getBalance() - amount);
        toAcc.setBalance(toAcc.getBalance() + amount);

        dao.updateAccount(fromAcc);
        dao.updateAccount(toAcc);

        System.out.println("Transferred $" + amount + " from " + fromId + " to " + toId);
    }
}

package com.example.bankapp;

import org.springframework.context.annotation.*;
import org.springframework.orm.hibernate5.HibernateTransactionManager;
import org.springframework.orm.hibernate5.LocalSessionFactoryBean;
import javax.sql.DataSource;
import org.springframework.jdbc.datasource.DriverManagerDataSource;
import java.util.Properties;

@Configuration
@EnableTransactionManagement
@ComponentScan(basePackages = "com.example.bankapp")
public class AppConfig {

    @Bean
    public DataSource dataSource() {
        DriverManagerDataSource ds = new DriverManagerDataSource();
        ds.setDriverClassName("com.mysql.cj.jdbc.Driver");
        ds.setUrl("jdbc:mysql://localhost:3306/testdb");
        ds.setUsername("root");
        ds.setPassword("yourpassword");
        return ds;
    }

    @Bean
    public LocalSessionFactoryBean sessionFactory() {
        LocalSessionFactoryBean factory = new LocalSessionFactoryBean();
        factory.setDataSource(dataSource());
        factory.setPackagesToScan("com.example.bankapp");

        Properties props = new Properties();
        props.put("hibernate.dialect", "org.hibernate.dialect.MySQL8Dialect");
        props.put("hibernate.hbm2ddl.auto", "update");
        props.put("hibernate.show_sql", "true");
        factory.setHibernateProperties(props);
        return factory;
    }

    @Bean
    public HibernateTransactionManager transactionManager() {
        HibernateTransactionManager txManager = new HibernateTransactionManager();
        txManager.setSessionFactory(sessionFactory().getObject());
        return txManager;
    }
}

package com.example.bankapp;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class MainApp {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext context =
                new AnnotationConfigApplicationContext(AppConfig.class);

        BankService bankService = context.getBean(BankService.class);

        try {
            bankService.transferMoney(101, 102, 500.0);
        } catch (Exception e) {
            System.out.println("Transaction failed: " + e.getMessage());
        }

        context.close();
    }
}
