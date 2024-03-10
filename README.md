%USAR EN SWI PROLOG

% Base de conocimientos

% Directiva para declarar el predicado cita/5 como dinámico
:- dynamic cita/5.

% Hechos: nombres y apellidos de los usuarios y los anfitriones
usuario("Juan Perez").
usuario("Ana Lopez").
usuario("Carlos Gomez").
anfitrion("Maria Gomez").
anfitrion("Pedro Sanchez").
anfitrion("Laura Ramirez").

% Hechos: fechas y horas de las citas ya programadas
% Formato: cita(Usuario, Anfitrion, (Dia, Mes, Ano), (Hora, Minuto), Canal)
cita("Juan Perez", "Maria Gomez", (22, 3, 2024), (10, 30), telefono).
cita("Ana Lopez", "Pedro Sanchez", (23, 3, 2024), (11, 0), chat).
cita("Carlos Gomez", "Laura Ramirez", (24, 3, 2024), (12, 0), email).

% Reglas: restricciones y condiciones para programar las citas

% Regla: el anfitrión está disponible si no tiene ninguna cita en la misma fecha y hora
disponible(A, F, H) :- anfitrion(A), fecha(F), hora(H), not(cita(_, A, F, H, _)).

% Regla: la fecha es válida si el día, el mes y el Ano son números enteros dentro de los rangos permitidos
fecha((D, M, A)) :- integer(D), integer(M), integer(A), D > 0, D =< 31, M > 0, M =< 12, A >= 2024.

% Regla: la hora es válida si la hora y el minuto son números enteros dentro de los rangos permitidos
hora((H, M)) :- integer(H), integer(M), H >= 0, H =< 23, M >= 0, M =< 59.

% Regla: el canal es válido si es uno de los siguientes: telefono, chat o email
canal(C) :- member(C, [telefono, chat, email]).

% Regla: hay un conflicto de horario si la cita solicitada se solapa con otra cita del mismo usuario o del mismo anfitrión
conflicto(U, A, F, (H, M)) :- cita(U, _, F, (H1, M1), _), % Hay otra cita del mismo usuario en la misma fecha
                              plus(H, 1, H2), % La hora de fin de la cita solicitada es una hora más que la hora de inicio
                              plus(M, 15, M2), % Los minutos de fin de la cita solicitada son 15 minutos más que los minutos de inicio
                              between(H1, H2, H), % La hora de inicio de la cita solicitada está entre la hora de inicio y la hora de fin de la otra cita
                              between(M1, M2, M). % Los minutos de inicio de la cita solicitada están entre los minutos de inicio y los minutos de fin de la otra cita
conflicto(U, A, F, (H, M)) :- cita(_, A, F, (H1, M1), _), % Hay otra cita del mismo anfitrión en la misma fecha
                              plus(H, 1, H2), % La hora de fin de la cita solicitada es una hora más que la hora de inicio
                              plus(M, 15, M2), % Los minutos de fin de la cita solicitada son 15 minutos más que los minutos de inicio
                              between(H1, H2, H), % La hora de inicio de la cita solicitada está entre la hora de inicio y la hora de fin de la otra cita
                              between(M1, M2, M). % Los minutos de inicio de la cita solicitada están entre los minutos de inicio y los minutos de fin de la otra cita

% Regla: la cita cumple con los requisitos si el usuario, el anfitrión, la fecha, la hora y el canal son válidos y no hay ningún conflicto de horario
cumple_requisitos(U, A, F, H, C) :- usuario(U), anfitrion(A), fecha(F), hora(H), canal(C), not(conflicto(U, A, F, H)).

% Interfaz de usuario

% Variable global para almacenar el nombre y apellido del usuario
:- dynamic nombre_usuario/1.

% Predicado principal que inicia el programa
inicio :- write("Bienvenido al asistente virtual de citas."), nl,
          write("Por favor, ingrese su nombre y apellido."), nl,
          read(Nombre), assert(nombre_usuario(Nombre)), nl,
          menu.

% Predicado que muestra el menú inicial y permite al usuario elegir una opción
menu :- write("¿Qué desea hacer?"), nl,
        write("1. Programar una cita."), nl,
        write("2. Consultar una cita."), nl,
        write("3. Cambiar de persona."), nl, % Nueva opción
        write("4. Salir del programa."), nl,
        read(Opcion), nl,
        ejecutar_opcion(Opcion).

% Predicado que ejecuta la opción elegida por el usuario
ejecutar_opcion(1) :- programar_cita.
ejecutar_opcion(2) :- consultar_cita.
ejecutar_opcion(3) :- change.
ejecutar_opcion(4) :- salir.
ejecutar_opcion(_) :- write("Opción inválida. Intente de nuevo."), nl, menu.


% Predicado que permite al usuario programar una cita
programar_cita :- write("Ingrese el nombre y apellido del anfitrión con quien desea agendar la cita."), nl,
                  read(Anfitrion), nl,
                  write("Ingrese la fecha deseada para la cita (DD, MM, AAAA)."), nl,
                  read((Dia, Mes, Ano)), nl,
                  write("Ingrese la hora deseada para la cita (HH, MM)."), nl,
                  read((Hora, Minuto)), nl,
                  write("Ingrese el canal de solicitud para la cita (telefono, chat o email)."), nl, % Aquí continué con el código
                  read(Canal), nl,
                  verificar_cita(Anfitrion, (Dia, Mes, Ano), (Hora, Minuto), Canal).

% Predicado que verifica si la cita solicitada cumple con los requisitos y la programa o informa del problema
verificar_cita(Anfitrion, Fecha, Hora, Canal) :- nombre_usuario(Usuario),
                                                 cumple_requisitos(Usuario, Anfitrion, Fecha, Hora, Canal),
                                                 assert(cita(Usuario, Anfitrion, Fecha, Hora, Canal)),
                                                 write("Su cita ha sido programada con éxito."), nl,
                                                 menu.
verificar_cita(Anfitrion, Fecha, Hora, Canal) :- nombre_usuario(Usuario),
                                                 conflicto(Usuario, Anfitrion, Fecha, Hora),
                                                 write("Lo sentimos, hay un conflicto de horario para la cita solicitada. Intente con otra fecha u hora."), nl,
                                                 menu.
verificar_cita(_, _, _, _) :- write("Lo sentimos, los datos ingresados no son válidos. Intente de nuevo."), nl,
                              menu.

% Predicado que permite al usuario consultar una cita
consultar_cita :- write("Ingrese el nombre y apellido del anfitrión con quien tiene la cita."), nl,
                  read(Anfitrion), nl,
                  mostrar_cita(Anfitrion).

% Predicado que muestra los detalles de la cita o informa si no hay ninguna
mostrar_cita(Anfitrion) :- nombre_usuario(Usuario),
                           bagof((Dia, Mes, Ano, Hora, Minuto, Canal), cita(Usuario, Anfitrion, (Dia, Mes, Ano), (Hora, Minuto), Canal), Citas), % Obtiene una lista con todas las citas que tiene el usuario con el anfitrión
                           write("Usted tiene las siguientes citas con "), write(Anfitrion), write(":"), nl,
                           mostrar_lista(Citas), % Muestra la lista de citas con sus detalles
                           menu.
mostrar_cita(Anfitrion) :- write("Usted no tiene una cita con "), write(Anfitrion), write("."), nl,
                           menu.

% Predicado que muestra los elementos de una lista
mostrar_lista([]) :- !. % Caso base: si la lista está vacía, termina
mostrar_lista(Citas) :- member((Dia, Mes, Ano, Hora, Minuto, Canal), Citas), % Obtiene un elemento de la lista, que es una cita
                        format("- El ~w/~w/~w a las ~w:~w por ~w.~n", [Dia, Mes, Ano, Hora, Minuto, Canal]), % Muestra el elemento con el formato deseado
                        delete(Citas, (Dia, Mes, Ano, Hora, Minuto, Canal), NuevaLista), % Elimina el elemento de la lista
                        mostrar_lista(NuevaLista). % Repite el proceso con la nueva lista

change:- write("Cambio de Persona."), nl,
         inicio.
% Predicado que termina el programa
salir :- write("Gracias por usar el asistente virtual de citas. Hasta pronto."), nl,
         abort.