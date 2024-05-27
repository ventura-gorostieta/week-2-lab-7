# week-2-lab-7
Laboratorio correspondiente a los temas de patrones de diseño

## 1.- Design Problem Solving:
**Scenario Description:** Participants are provided with a series of common software design challenges. They will need to choose appropriate design patterns to solve these specific problems effectively.
Design Challenges:
* **Global Configuration Management:** Design a system that ensures a single, globally accessible configuration object without access conflicts.

```
public class GlobalConfig {
    private static GlobalConfig instance;
    private Properties properties;

    private GlobalConfig() {
        properties = new Properties();
        // Cargar la configuración necesaria
    }

/**
 * Si la instancia del objeto no existe, se crea, de lo contrario, se retorna la instancia existente.
 */
    public static synchronized GlobalConfig getInstance() {
        if (instance == null) {
            instance = new GlobalConfig();
        }
        return instance;
    }

    public String getProperty(String key) {
        return properties.getProperty(key);
    }

    public void setProperty(String key, String value) {
        properties.setProperty(key, value);
    }
}


```


* **Dynamic Object Creation Based on User Input:** Implement a system to dynamically create various types of user interface elements based on user actions.

```
/**
 * Interfaz que define el comportamiento para los tipos de documentos: open
 */
public interface Document {
    void open();
}

public class PDFDocument implements Document {
    public void open() {
        System.out.println("Opening PDF Document");
    }
}

public class WordDocument implements Document {
    public void open() {
        System.out.println("Opening Word Document");
    }
}

public class ExcelDocument implements Document {
    public void open() {
        System.out.println("Opening Excel Document");
    }
}


/**
 * Este metodo main, controla la fabrica de objetos,
 * es decir, acorde al parametro que se le pase, creará un tipo de documento a abrir
 */
public class DocumentFactory {
    public static Document createDocument(String type) {
        switch (type.toLowerCase()) {
            case "pdf":
                return new PDFDocument();
            case "word":
                return new WordDocument();
            case "excel":
                return new ExcelDocument();
            default:
                throw new IllegalArgumentException("Unknown document type");
        }
    }
}


public class FactoryPatternExample {
    public static void main(String[] args) {
        Document pdf = DocumentFactory.createDocument("pdf");
        Document word = DocumentFactory.createDocument("word");
        Document excel = DocumentFactory.createDocument("excel");

        pdf.open();
        word.open();
        excel.open();
    }
}

```

* **State Change Notification Across System Components:** Ensure components are notified about changes in the state of other parts without creating tight coupling.

```
/**
 * Define una interfaz para ser notificado sobre cambios en el sujeto.
 */
public interface Observer {
    void update(String eventType, String data);
}

/**
 * Mantiene una lista de observadores y proporciona métodos para agregar:
 *  eliminar y notificar a los observadores.
 */
public interface Subject {
    void attach(Observer observer);
    void detach(Observer observer);
    void notifyObservers(String eventType, String data);
}

/**
 * Implementa la interfaz Subject y mantiene el estado de interés para los observadores.
 */
public class EventManager implements Subject {
    private List<Observer> observers = new ArrayList<>();

    public void attach(Observer observer) {
        observers.add(observer);
    }

    public void detach(Observer observer) {
        observers.remove(observer);
    }

    public void notifyObservers(String eventType, String data) {
        for (Observer observer : observers) {
            observer.update(eventType, data);
        }
    }
}

/**
 * Implementa la interfaz Observer y actualiza su estado en respuesta a las notificaciones del sujeto.
 */
public class ConcreteObserver implements Observer {
    private String name;

    public ConcreteObserver(String name) {
        this.name = name;
    }

    public void update(String eventType, String data) {
        System.out.println(name + " received " + eventType + " with data: " + data);
    }
}

public class ObserverPatternExample {
    public static void main(String[] args) {
        // Crear el sujeto
        EventManager eventManager = new EventManager();
        // Creamos los observadores
        ConcreteObserver observer1 = new ConcreteObserver("Observer1");
        ConcreteObserver observer2 = new ConcreteObserver("Observer2");

        // Registrar los observadores en el sujeto
        eventManager.attach(observer1);
        eventManager.attach(observer2);

        // Notificamos a los observadores de diferentes eventos
        eventManager.notifyObservers("EVENT_TYPE_1", "data 1");
        eventManager.notifyObservers("EVENT_TYPE_2", "data 2");

        // Se quita a un observador de la lista de eventos.
        eventManager.detach(observer1);
        // Notificamos a los observadores restantes
        eventManager.notifyObservers("EVENT_TYPE_3", "data 3");
    }
}

```


* **Efficient Management of Asynchronous Operations:** Manage multiple asynchronous operations like API calls which need to be coordinated without blocking the main application workflow.

```
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.Executors;

public class AsyncManager {
    private final ExecutorService executor;

    public AsyncManager(ThreadPoolExecutor executor) {
        this.executor = executor;
    }

    public CompletableFuture<String> performAsyncOperation() {
        return CompletableFuture.supplyAsync(() -> {
            // Simular operación asíncrona, por ejemplo, llamada APIs
            try {
                Thread.sleep(1000); // Simula una operación larga en ms
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return "Result from async operation";
        }, executor);
    }

    public void shutdown() {
        executor.shutdown();
        try {
            if (!executor.awaitTermination(60, TimeUnit.SECONDS)) {
                executor.shutdownNow();
                if (!executor.awaitTermination(60, TimeUnit.SECONDS)) {
                    System.err.println("Executor did not terminate");
                }
            }
        } catch (InterruptedException ie) {
            executor.shutdownNow();
            Thread.currentThread().interrupt();
        }
    }
}

/**
 * Metodo main, donde se configura un threadPoolExecutor
 * El cual puede ser importante para cuidar el no rebasar los 
 * recursos de hardware al momento de lanzar hilos en segundo plano.
 * se simular 3 promesas y se lanzan a ejecución, simulando llamadas no bloqueantes.
 */
public class AsyncPatternExample {
    public static void main(String[] args) {
        ThreadPoolExecutor executor = new ThreadPoolExecutor(
                4, // core pool size
                8, // maximum pool size
                60L, // idle thread keep-alive time
                TimeUnit.SECONDS,
                new ArrayBlockingQueue<>(100) // work queue
        );
        
        AsyncManager asyncManager = new AsyncManager(executor);

        CompletableFuture<String> future1 = asyncManager.performAsyncOperation();
        CompletableFuture<String> future2 = asyncManager.performAsyncOperation();
        CompletableFuture<String> future3 = asyncManager.performAsyncOperation();

        future1.thenAccept(result -> System.out.println("Received async result 1: " + result));
        future2.thenAccept(result -> System.out.println("Received async result 2: " + result));
        future3.thenAccept(result -> System.out.println("Received async result 3: " + result));

        System.out.println("Main thread continues to run...");

        // Esperar a que todas las tareas se completen antes de cerrar el executor
        CompletableFuture.allOf(future1, future2, future3).join();
        asyncManager.shutdown();
    }
}


```
**Task:** Outline solutions that integrate these patterns into a cohesive design to address the challenges.

## Project Execution Simulation:

Para la simulacoón solicitada se hace uso de los patrones de diseño Singleton, Factory, Observer y Async para resolver los desafíos de diseño especificados. El proyecto simulado es una aplicación de gestión de documentos que requiere configuración global (singleton), creación dinámica de documentos(factory), notificación de cambios de estado(observer) y gestión de operaciones asíncronas(asyn thread).

### Enfoque y Justificación

**Singleton para la Gestión de Configuración Global:**
**Enfoque:** Usar el patrón Singleton para asegurar que la configuración global del sistema sea accesible y consistente en todo momento.
**Justificación:** El patrón Singleton garantiza una única instancia de la configuración que es compartida por todos los componentes del sistema, evitando conflictos de acceso y manteniendo la coherencia de la configuración.

**Factory para la Creación Dinámica de Documentos:**

**Enfoque:** Utilizar el patrón Factory para crear diferentes tipos de documentos (PDF, Word, Excel) basados en parametros de entrada.
**Justificación:** El patrón Factory permite la creación de objetos sin especificar la clase exacta del objeto que se va a crear, facilitando la extensión del sistema para soportar nuevos tipos de documentos sin modificar el código existente.

**Observer para la Notificación de Cambios de Estado:**

**Enfoque:** Implementar el patrón Observer para notificar a los componentes interesados sobre cambios en el estado de otros componentes sin crear un acoplamiento fuerte.
**Justificación:** El patrón Observer facilita la comunicación entre componentes, permitiendo que los observadores reaccionen a los cambios de estado de manera eficiente y desacoplada.

**Async para la Gestión de Operaciones Asíncronas:**

**Enfoque:** Utilizar el patrón Async junto con un ThreadPoolExecutor para manejar operaciones asíncronas, como llamadas a APIs externas, sin bloquear el flujo principal de la aplicación.
**Justificación:** El patrón Async mejora la capacidad de respuesta del sistema al permitir que las operaciones largas se ejecuten en segundo plano, mientras que el ThreadPoolExecutor gestiona eficientemente los recursos del sistema.

### Simulación del proyecto, tomando como base las clases y objetos del paso 1.

```
public class ProjectExecutionSimulation {
    public static void main(String[] args) {
        // Configuración global usando Singleton, crea una unica instancia
        ConfigurationManager config = ConfigurationManager.getInstance();
        config.setProperty("app.name", "Document Management System");

        // Creación dinámica de documentos usando Factory, retorna el tipo de objeto según el parametro.
        Document pdf = DocumentFactory.createDocument("pdf");
        Document word = DocumentFactory.createDocument("word");
        Document excel = DocumentFactory.createDocument("excel");

        pdf.open();
        word.open();
        excel.open();

        // Notificación de cambios de estado usando Observer
        EventManager eventManager = new EventManager();
        ConcreteObserver observer1 = new ConcreteObserver("Observer1");
        ConcreteObserver observer2 = new ConcreteObserver("Observer2");
        // se han generado 2 suscriptores al observer.
        eventManager.attach(observer1);
        eventManager.attach(observer2);

        eventManager.notifyObservers("EVENT_TYPE_1", "Document opened");
        eventManager.notifyObservers("EVENT_TYPE_2", "Document saved");

        eventManager.detach(observer1);
        eventManager.notifyObservers("EVENT_TYPE_3", "Document closed");

        // Gestión de operaciones asíncronas usando Async y ThreadPoolExecutor
        // Gestionamos mejor los recursos e hilos usando threadPoolExecutor.
        ThreadPoolExecutor executor = new ThreadPoolExecutor(
                4, 
                8, 
                60L, 
                TimeUnit.SECONDS,
                new ArrayBlockingQueue<>(100)
        );

        AsyncManager asyncManager = new AsyncManager(executor);

// se lanzan 3 tareas o simulacion de llamadas, usando async, se van a segundo plano
        CompletableFuture<String> future1 = asyncManager.performAsyncOperation();
        CompletableFuture<String> future2 = asyncManager.performAsyncOperation();
        CompletableFuture<String> future3 = asyncManager.performAsyncOperation();

        future1.thenAccept(result -> System.out.println("Received async result 1: " + result));
        future2.thenAccept(result -> System.out.println("Received async result 2: " + result));
        future3.thenAccept(result -> System.out.println("Received async result 3: " + result));

        System.out.println("Main thread continues to run...");

        // Usamos allOd para esperar a que todas las tareas se completen
        CompletableFuture.allOf(future1, future2, future3).join();
        asyncManager.shutdown();
    }
}

```


