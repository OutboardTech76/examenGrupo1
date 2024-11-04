# Examen Prácticas Metodologías Ágiles del Desarrollo del Software

El examen consiste en implementar varias funcionalidades sobre el proyecto de todo-list que se os proporciona en el siguiente repositorio de github (https://classroom.github.com/a/uBi5kp5u). Como en las prácticas realizadas en el aula, debes unirte a dicho repositorio para obtener el código de partida del examen. Además, para facilitar la implementación de dichas funcionalidades, se os proporcionan una serie de tests para que podáis probar vuestro código.

La duración del examen es de 1 hora y 30 minutos y supone un 20% sobre la nota final de la asignatura. Cada test especifica su puntuación sobre la nota del examen.

La entrega se realizará por Moodle en un fichero .zip tras haber ejecutado `./mvnw clean`, la carpeta .git debe encontrarse en el fichero comprimido. Además, todo ha de subirse al repositorio de GitHub facilitado, de manera similar a lo realizado en las prácticas. En este caso ha de crearse un nuevo issue para cada funcionalidad, una nueva rama y añadirle la etiqueta *Examen grupo 1*, no es necesario vincular ningún milestone ni proyecto.

Los tests mostrados pueden modificarse para añadir información en caso de ser necesario, en ningún caso se puede eliminar código de ellos.

## Funcionalidad 1: Prioridad de tareas
Esta funcionalidad nos permite añadir un campo de prioridad a las tareas. Estas prioridades son 'Alta', 'Media' y 'Baja'. Las prioridades deben de añadirse al crear una nueva tarea además de visualizarse en el listado.

Test en `TareaServiceTest` (2 puntos):
```
@Test
public void testCrearTareaConPrioridad() {
    // GIVEN
    Long usuarioId = addUsuarioTareasBD().get("usuarioId");

    // WHEN
    TareaData tareaAlta = tareaService.nuevaTareaUsuario(usuarioId, "Estudiar", "Alta");
    TareaData tareaBaja = tareaService.nuevaTareaUsuario(usuarioId, "Comprar material", "Baja");

    // THEN
    assertThat(tareaAlta.getPrioridad()).isEqualTo("Alta");
    assertThat(tareaBaja.getPrioridad()).isEqualTo("Baja");
}
```
Test en `TareaWebTest` (1.5 puntos):
```
@Test
public void testListadoDeTareasConPrioridad() throws Exception {
    // GIVEN
    Long usuarioId = addUsuarioTareasBD().get("usuarioId");
    when(managerUserSession.usuarioLogeado()).thenReturn(usuarioId);

    tareaService.nuevaTareaUsuario(usuarioId, "Estudiar", "Alta");
    tareaService.nuevaTareaUsuario(usuarioId, "Hacer ejercicio", "Media");

    // WHEN, THEN
    this.mockMvc.perform(get("/usuarios/" + usuarioId + "/tareas"))
            .andExpect(content().string(containsString("Alta")))
            .andExpect(content().string(containsString("Media")));
}
```
## Funcionalidad 2: Búsqueda de tareas
Esta funcionalidad permite realizar un filtrado en las tareas que se desean mostrar. Para ello se realiza un filtro por nombre, de manera que si el nombre coincide, la tarea es mostrada. Recordad que una url puede recibir parámetros añadiéndolos al enlace de la forma `localhost:8080/prueba?numero=2` esto envía el valor 2 a la variable numero, en código se trata de un RequestParam.

Test en `TareaServiceTest` (3 puntos):
```
@Test
public void testBuscarTareasPorTitulo() {
    // GIVEN
    Long usuarioId = addUsuarioTareasBD().get("usuarioId");
    tareaService.nuevaTareaUsuario(usuarioId, "Estudiar examen");
    tareaService.nuevaTareaUsuario(usuarioId, "Comprar material");

    // WHEN
    List<TareaData> resultados = tareaService.buscarTareasPorTitulo(usuarioId, "examen");

    // THEN
    assertThat(resultados).hasSize(1);
    assertThat(resultados.get(0).getTitulo()).isEqualTo("Estudiar examen");
}

```
Test en `TareaWebTest` (1.5 puntos):
```
@Test
public void testBuscarTareasEnWeb() throws Exception {
    // GIVEN
    Long usuarioId == addUsuarioTareasBD().get("usuarioId");
    when(managerUserSession.usuarioLogeado()).thenReturn(usuarioId);

    tareaService.nuevaTareaUsuario(usuarioId, "Leer libro");
    tareaService.nuevaTareaUsuario(usuarioId, "Leer noticias");

    // WHEN, THEN
    this.mockMvc.perform(get("/usuarios/" + usuarioId + "/tareas").param("search", "libro"))
            .andExpect(content().string(containsString("Leer libro")))
            .andExpect(content().string(not(containsString("Leer noticias"))));
}
```

## Funcionalidad 3: Comprobar excepciones
Esta funcionalidad obliga que se lancen excepciones en los métodos de servicio cuando las tareas no existen.

Test en `TareaServiceTest` (2 puntos):
```
import static org.assertj.core.api.AssertionsForClassTypes.assertThatThrownBy;

@Test
public void comprobarExcepciones() {
    assertThatThrownBy(() -> tareaService.modificaTarea(1L, "Tarea 1"))
            .isInstanceOf(TareaServiceException.class);
    assertThatThrownBy(() -> tareaService.borraTarea(1L))
            .isInstanceOf(TareaServiceException.class);
    assertThatThrownBy(() -> tareaService.usuarioContieneTarea(1L, 1L))
            .isInstanceOf(TareaServiceException.class);
}

```

