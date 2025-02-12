package Controller;

import Model.modelDiary;
import Vista.DiaryProgram;
import javax.swing.JOptionPane;
import org.bson.Document;
import java.text.SimpleDateFormat;

// Clase concreta para Template Method
public class ActualizarAmigoAction extends DiaryAction {

    public ActualizarAmigoAction(modelDiary model, DiaryProgram view) {
        super(model, view);
    }

    @Override
    protected void doExecute() {
        int filaSeleccionada = view.txtDatos.getSelectedRow();
        if (filaSeleccionada >= 0) {
            String nombreApodo = (String) view.txtDatos.getValueAt(filaSeleccionada, 1);

            if (validarCampos()) {
                String genero = view.rdFemenino.isSelected() ? "Femenino" : "Masculino";
                SimpleDateFormat formatoFecha = new SimpleDateFormat("dd/MM/yyyy");
                String fecha = formatoFecha.format(view.jDFecha.getDate());
                String numero = view.txtNumeroTelefono.getText();
                String comida = view.txtComidaFavorita.getText();
                int edad = (int) view.spinEdad.getValue();
                String color = (String) view.cboxColor.getSelectedItem();
                String musica = view.listMusica.getSelectedValue();
                String importante = view.txtImportante.getText();

                if (edad < 10 || edad > 65) {
                    JOptionPane.showMessageDialog(view, "La edad debe ser entre 10 y 65 años.");
                    return;
                }

                Document nuevosDatos = new Document("genero", genero)
                        .append("fecha", fecha)
                        .append("numero", numero)
                        .append("comida", comida)
                        .append("edad", edad)
                        .append("color", color)
                        .append("musica", musica)
                        .append("importante", importante);

                model.actualizarAmigo(nombreApodo, nuevosDatos);
                limpiarCampos();
                mostrarAmigosEnTabla();
            }
        } else {
            JOptionPane.showMessageDialog(view, "Por favor, selecciona un amigo de la tabla para actualizar.");
        }
    }

    private boolean validarCampos() {
        // Validación de campos
        boolean esValido = true;
        if (view.txtNombre.getText().trim().isEmpty()) {
            esValido = false;
            JOptionPane.showMessageDialog(view, "El nombre/apodo no puede estar vacío.");
        }
        if (view.jDFecha.getDate() == null) {
            esValido = false;
            JOptionPane.showMessageDialog(view, "La fecha de nacimiento no puede estar vacía.");
        }
        if (view.txtNumeroTelefono.getText().trim().isEmpty()) {
            esValido = false;
            JOptionPane.showMessageDialog(view, "El número de teléfono no puede estar vacío.");
        }
        if (view.txtComidaFavorita.getText().trim().isEmpty()) {
            esValido = false;
            JOptionPane.showMessageDialog(view, "La comida favorita no puede estar vacía.");
        }
        if (view.spinEdad.getValue() == null || (int) view.spinEdad.getValue() < 10 || (int) view.spinEdad.getValue() > 65) {
            esValido = false;
            JOptionPane.showMessageDialog(view, "La edad debe estar entre 10 y 65 años.");
        }
        if (view.cboxColor.getSelectedItem() == null) {
            esValido = false;
            JOptionPane.showMessageDialog(view, "Debes seleccionar un color favorito.");
        }
        if (view.listMusica.getSelectedValue() == null) {
            esValido = false;
            JOptionPane.showMessageDialog(view, "Debes seleccionar un género musical favorito.");
        }
        if (view.txtImportante.getText().trim().isEmpty()) {
            esValido = false;
            JOptionPane.showMessageDialog(view, "El campo de información importante no puede estar vacío.");
        }
        return esValido;
    }

    private void limpiarCampos() {
        // Limpia los campos del formulario
        view.txtNombre.setText("");
        view.jDFecha.setDate(null);
        view.txtNumeroTelefono.setText("");
        view.txtComidaFavorita.setText("");
        view.spinEdad.setValue(0);
        view.cboxColor.setSelectedIndex(0);
        view.listMusica.clearSelection();
        view.txtImportante.setText("");
        view.buttonGroup1.clearSelection();
    }

    private void mostrarAmigosEnTabla() {
        // Muestra los amigos en la tabla
        view.txtDatos.setModel(new javax.swing.table.DefaultTableModel(
            new Object [][] {},
            new String [] {
                "Género", "Nombre/Apodo", "Fecha de Nacimiento", "Número de Teléfono", "Comida Favorita", "Edad", "Color Favorito", "Género Musical", "Información Importante"
            }
        ));
        for (Document amigo : model.obtenerAmigos()) {
            ((javax.swing.table.DefaultTableModel) view.txtDatos.getModel()).addRow(new Object[]{
                amigo.getString("genero"),
                amigo.getString("nombreApodo"),
                amigo.getString("fecha"),
                amigo.getString("numero"),
                amigo.getString("comida"),
                amigo.getInteger("edad"),
                amigo.getString("color"),
                amigo.getString("musica"),
                amigo.getString("importante")
            });
        }
    }
}

package Controller;

import Model.modelSchedule;
import Vista.scheduleOutings;
import javax.swing.JOptionPane;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.time.LocalDate;
import java.time.ZoneId;
import org.bson.Document;
import javax.swing.table.DefaultTableModel;
import java.util.List;

// Command concreto
public class AgendarSalidaCommand implements ScheduleCommand {
    private modelSchedule model;
    private scheduleOutings view;

    public AgendarSalidaCommand(modelSchedule model, scheduleOutings view) {
        this.model = model;
        this.view = view;
    }

    @Override
    public void execute() {
        limpiarErrores();

        String nombreSalida = view.txtNombreSalida.getText().trim();
        Date fechaSalidaDate = view.jDFechaSalida.getDate();
        String lugarSalida = view.txtLugar.getText().trim();
        int filaSeleccionada = view.tblDatosSalidas.getSelectedRow();

        boolean hayErrores = false;

        if (nombreSalida.isEmpty()) {
            view.lblErrorNombreSalida.setText("El nombre de la salida es obligatorio.");
            hayErrores = true;
        }

        if (fechaSalidaDate == null) {
            view.lblErrorFechaSalida.setText("La fecha de la salida es obligatoria.");
            hayErrores = true;
        } else {
            LocalDate today = LocalDate.now();
            LocalDate fechaSalida = fechaSalidaDate.toInstant().atZone(ZoneId.systemDefault()).toLocalDate();

            if (fechaSalida.isBefore(today)) {
                view.lblErrorFechaSalida.setText("La fecha de la salida no puede ser en el pasado.");
                hayErrores = true;
            } else {
                view.lblErrorFechaSalida.setText("");
            }
        }

        if (lugarSalida.isEmpty()) {
            view.lblErrorLugarSalida.setText("El lugar de la salida es obligatorio.");
            hayErrores = true;
        }

        if (filaSeleccionada == -1) {
            JOptionPane.showMessageDialog(view, "Por favor, selecciona con quién vas a salir.", "Error", JOptionPane.ERROR_MESSAGE);
            hayErrores = true;
        }

        if (!hayErrores) {
            List<Document> salidasExistentes = model.buscarSalidaPorNombre(nombreSalida);

            if (!salidasExistentes.isEmpty()) {
                JOptionPane.showMessageDialog(view, "Ya existe una salida con el mismo nombre. Por favor elige un nombre diferente.", "Error", JOptionPane.ERROR_MESSAGE);
                return;
            }

            String amigoSeleccionado = (String) view.tblDatosSalidas.getValueAt(filaSeleccionada, 1);

            SimpleDateFormat formatoFecha = new SimpleDateFormat("dd/MM/yyyy");
            String fechaSalida = formatoFecha.format(fechaSalidaDate);

            boolean agendado = model.agendarSalida(amigoSeleccionado, nombreSalida, fechaSalida, lugarSalida);
            if (agendado) {
                String mensaje = "Has agendado una salida con: " + amigoSeleccionado + "\n"
                        + "Nombre de la salida: " + nombreSalida + "\n"
                        + "Fecha: " + fechaSalida + "\n"
                        + "Lugar: " + lugarSalida;

                JOptionPane.showMessageDialog(view, mensaje, "Resumen de la salida", JOptionPane.INFORMATION_MESSAGE);
                cargarSalidasEnTabla();
            } else {
                JOptionPane.showMessageDialog(view, "Error al agendar la salida.", "Error", JOptionPane.ERROR_MESSAGE);
            }
        }
    }

    private void limpiarErrores() {
        view.lblErrorNombreSalida.setText("");
        view.lblErrorFechaSalida.setText("");
        view.lblErrorLugarSalida.setText("");
    }

    private void cargarSalidasEnTabla() {
        DefaultTableModel modeloTablaSalidas = (DefaultTableModel) view.tblSalidas.getModel();
        modeloTablaSalidas.setRowCount(0);

        List<Document> salidas = model.obtenerSalidas();
        for (Document salida : salidas) {
            modeloTablaSalidas.addRow(new Object[]{
                salida.getString("amigo"),
                salida.getString("nombreSalida"),
                salida.getString("fecha"),
                salida.getString("lugar")
            });
        }
    }
}

package Controller;

import Model.modelDiary;
import Vista.DiaryProgram;
import javax.swing.JOptionPane;
import org.bson.Document;
import java.text.SimpleDateFormat;
import java.util.List;

// Clase concreta para Template Method
public class AgregarAmigoAction extends DiaryAction {

    public AgregarAmigoAction(modelDiary model, DiaryProgram view) {
        super(model, view);
    }

    @Override
    protected void doExecute() {
        if (validarCampos()) {
            String genero = view.rdFemenino.isSelected() ? "Femenino" : "Masculino";
            String nombreApodo = view.txtNombre.getText();
            SimpleDateFormat formatoFecha = new SimpleDateFormat("dd/MM/yyyy");
            String fecha = formatoFecha.format(view.jDFecha.getDate());
            String numero = view.txtNumeroTelefono.getText();
            String comida = view.txtComidaFavorita.getText();
            int edad = (int) view.spinEdad.getValue();
            String color = (String) view.cboxColor.getSelectedItem();
            String musica = view.listMusica.getSelectedValue();
            String importante = view.txtImportante.getText();

            if (edad < 10 || edad > 65) {
                view.lblErrorEdad.setText("La edad debe ser entre 10 y 65 años.");
                return;
            }

            List<Document> amigosExistentes = model.buscarAmigo(new Document("nombreApodo", nombreApodo));
            if (!amigosExistentes.isEmpty()) {
                view.lblErrorNombre.setText("Ya existe un amigo con el mismo nombre/apodo.");
                return;
            }

            model.agregarAmigo(genero, nombreApodo, fecha, numero, comida, edad, color, musica, importante);
            JOptionPane.showMessageDialog(view, "Amigo guardado correctamente.", "Amigo Guardado", JOptionPane.INFORMATION_MESSAGE);

            model.obtenerDatos();
            limpiarCampos();
            mostrarAmigosEnTabla();
        }
    }

    private boolean validarCampos() {
        // Validación de campos
        boolean esValido = true;
        limpiarErrores();
        if (view.txtNombre.getText().trim().isEmpty()) {
            esValido = false;
            view.lblErrorNombre.setText("El nombre/apodo no puede estar vacío.");
        }
        if (view.jDFecha.getDate() == null) {
            esValido = false;
            view.lblErrorNacimiento.setText("La fecha de nacimiento no puede estar vacía.");
        }
        if (view.txtNumeroTelefono.getText().trim().isEmpty()) {
            esValido = false;
            view.lblErrorNumero.setText("El número de teléfono no puede estar vacío.");
        }
        if (view.txtComidaFavorita.getText().trim().isEmpty()) {
            esValido = false;
            view.lblErrorComidaFavorita.setText("La comida favorita no puede estar vacía.");
        }
        if (view.spinEdad.getValue() == null || (int) view.spinEdad.getValue() < 10 || (int) view.spinEdad.getValue() > 65) {
            esValido = false;
            view.lblErrorEdad.setText("La edad debe estar entre 10 y 65 años.");
        }
        if (view.cboxColor.getSelectedItem() == null) {
            esValido = false;
            view.lblErrorColor.setText("Debes seleccionar un color favorito.");
        }
        if (view.listMusica.getSelectedValue() == null) {
            esValido = false;
            view.lblErrorGeneroMusical.setText("Debes seleccionar un género musical favorito.");
        }
        if (view.txtImportante.getText().trim().isEmpty()) {
            esValido = false;
            view.lblErrorAcontecimiento.setText("El campo de información importante no puede estar vacío.");
        }
        return esValido;
    }

    private void limpiarErrores() {
        view.lblErrorNombre.setText("");
        view.lblErrorNacimiento.setText("");
        view.lblErrorNumero.setText("");
        view.lblErrorComidaFavorita.setText("");
        view.lblErrorEdad.setText("");
        view.lblErrorColor.setText("");
        view.lblErrorGeneroMusical.setText("");
        view.lblErrorAcontecimiento.setText("");
    }

    private void limpiarCampos() {
        // Limpia los campos del formulario
        view.txtNombre.setText("");
        view.jDFecha.setDate(null);
        view.txtNumeroTelefono.setText("");
        view.txtComidaFavorita.setText("");
        view.spinEdad.setValue(0);
        view.cboxColor.setSelectedIndex(0);
        view.listMusica.clearSelection();
        view.txtImportante.setText("");
        view.buttonGroup1.clearSelection();
        limpiarErrores();
    }

    private void mostrarAmigosEnTabla() {
        // Muestra los amigos en la tabla
        view.txtDatos.setModel(new javax.swing.table.DefaultTableModel(
            new Object [][] {},
            new String [] {
                "Género", "Nombre/Apodo", "Fecha de Nacimiento", "Número de Teléfono", "Comida Favorita", "Edad", "Color Favorito", "Género Musical", "Información Importante"
            }
        ));
        for (Document amigo : model.obtenerAmigos()) {
            ((javax.swing.table.DefaultTableModel) view.txtDatos.getModel()).addRow(new Object[]{
                amigo.getString("genero"),
                amigo.getString("nombreApodo"),
                amigo.getString("fecha"),
                amigo.getString("numero"),
                amigo.getString("comida"),
                amigo.getInteger("edad"),
                amigo.getString("color"),
                amigo.getString("musica"),
                amigo.getString("importante")
            });
        }
    }
}

package Controller;

import Model.modelDiary;
import Vista.DiaryProgram;
import javax.swing.JOptionPane;
import javax.swing.table.DefaultTableModel;
import org.bson.Document;
import java.util.List;

// Clase concreta para Template Method
public class BuscarAmigoAction extends DiaryAction {

    public BuscarAmigoAction(modelDiary model, DiaryProgram view) {
        super(model, view);
    }

    @Override
    protected void doExecute() {
        String criterio = view.txtBuscar.getText().trim();
        if (criterio.isEmpty()) {
            JOptionPane.showMessageDialog(view, "Por favor, ingresa un criterio de búsqueda.");
            return;
        }

        Document filtro = new Document("$or", List.of(
            new Document("nombreApodo", new Document("$regex", criterio).append("$options", "i")),
            new Document("genero", new Document("$regex", criterio).append("$options", "i")),
            new Document("numero", new Document("$regex", criterio).append("$options", "i"))
        ));

        DefaultTableModel modeloTabla = (DefaultTableModel) view.txtDatos.getModel();
        modeloTabla.setRowCount(0);

        boolean encontrado = false;
        for (Document amigo : model.buscarAmigo(filtro)) {
            modeloTabla.addRow(new Object[]{
                amigo.getString("genero"),
                amigo.getString("nombreApodo"),
                amigo.getString("fecha"),
                amigo.getString("numero"),
                amigo.getString("comida"),
                amigo.getInteger("edad"),
                amigo.getString("color"),
                amigo.getString("musica"),
                amigo.getString("importante")
            });
            encontrado = true;
        }
        if (encontrado) {
            JOptionPane.showMessageDialog(view, "Amigo encontrado.");
        } else {
            JOptionPane.showMessageDialog(view, "No se encontraron amigos con ese criterio.");
        }
    }
}

package Controller;

import Model.modelSchedule;
import Vista.scheduleOutings;
import javax.swing.JOptionPane;
import org.bson.Document;
import java.util.List;

// Command concreto
public class BuscarSalidaCommand implements ScheduleCommand {
    private modelSchedule model;
    private scheduleOutings view;

    public BuscarSalidaCommand(modelSchedule model, scheduleOutings view) {
        this.model = model;
        this.view = view;
    }

    @Override
    public void execute() {
        String nombreBusqueda = view.txtBuscarAmigoSalida.getText();

        if (nombreBusqueda.isEmpty()) {
            JOptionPane.showMessageDialog(view, "Por favor ingrese el nombre de la salida.");
            return;
        }

        List<Document> resultados = model.buscarSalidaPorNombre(nombreBusqueda);

        if (resultados.isEmpty()) {
            JOptionPane.showMessageDialog(view, "No se encontraron salidas con ese nombre.");
        } else {
            StringBuilder mensaje = new StringBuilder("Resultados encontrados:\n");
            for (Document salida : resultados) {
                mensaje.append("Amigo: ").append(salida.getString("amigo")).append("\n")
                       .append("Salida: ").append(salida.getString("nombreSalida")).append("\n")
                       .append("Fecha: ").append(salida.getString("fecha")).append("\n")
                       .append("Lugar: ").append(salida.getString("lugar")).append("\n\n");
            }
            JOptionPane.showMessageDialog(view, mensaje.toString());
        }
    }
}

package Controller;

import Model.modelDiary;
import Vista.DiaryProgram;

// Template Method
public abstract class DiaryAction {

    protected modelDiary model;
    protected DiaryProgram view;

    public DiaryAction(modelDiary model, DiaryProgram view) {
        this.model = model;
        this.view = view;
    }

    public final void execute() {
        preExecute();
        doExecute();
        postExecute();
    }

    protected void preExecute() {
        // Pasos comunes antes de la acción específica
    }

    protected abstract void doExecute();

    protected void postExecute() {
        // Pasos comunes después de la acción específica
    }
}

package Controller;

import Model.modelDiary;
import Vista.DiaryProgram;
import javax.swing.JOptionPane;
import org.bson.Document;

// Clase concreta para Template Method
public class EliminarAmigoAction extends DiaryAction {

    public EliminarAmigoAction(modelDiary model, DiaryProgram view) {
        super(model, view);
    }

    @Override
    protected void doExecute() {
        int filaSeleccionada = view.txtDatos.getSelectedRow();
        if (filaSeleccionada >= 0) {
            String nombreApodo = (String) view.txtDatos.getValueAt(filaSeleccionada, 1);
            model.eliminarAmigo(nombreApodo);
            mostrarAmigosEnTabla();
            limpiarCampos();
        } else {
            JOptionPane.showMessageDialog(view, "Por favor, selecciona un amigo de la tabla para eliminar.");
        }
    }

    private void limpiarCampos() {
        // Limpia los campos del formulario
        view.txtNombre.setText("");
        view.jDFecha.setDate(null);
        view.txtNumeroTelefono.setText("");
        view.txtComidaFavorita.setText("");
        view.spinEdad.setValue(0);
        view.cboxColor.setSelectedIndex(0);
        view.listMusica.clearSelection();
        view.txtImportante.setText("");
        view.buttonGroup1.clearSelection();
    }

    private void mostrarAmigosEnTabla() {
        // Muestra los amigos en la tabla
        view.txtDatos.setModel(new javax.swing.table.DefaultTableModel(
            new Object [][] {},
            new String [] {
                "Género", "Nombre/Apodo", "Fecha de Nacimiento", "Número de Teléfono", "Comida Favorita", "Edad", "Color Favorito", "Género Musical", "Información Importante"
            }
        ));
        for (Document amigo : model.obtenerAmigos()) {
            ((javax.swing.table.DefaultTableModel) view.txtDatos.getModel()).addRow(new Object[]{
                amigo.getString("genero"),
                amigo.getString("nombreApodo"),
                amigo.getString("fecha"),
                amigo.getString("numero"),
                amigo.getString("comida"),
                amigo.getInteger("edad"),
                amigo.getString("color"),
                amigo.getString("musica"),
                amigo.getString("importante")
            });
        }
    }
}

package Controller;

import Model.modelSchedule;
import Vista.scheduleOutings;
import javax.swing.JOptionPane;
import javax.swing.table.DefaultTableModel;
import org.bson.Document;
import java.util.List;

// Command concreto
public class EliminarSalidaCommand implements ScheduleCommand {
    private modelSchedule model;
    private scheduleOutings view;

    public EliminarSalidaCommand(modelSchedule model, scheduleOutings view) {
        this.model = model;
        this.view = view;
    }

    @Override
    public void execute() {
        int filaSeleccionada = view.tblSalidas.getSelectedRow();

        if (filaSeleccionada == -1) {
            JOptionPane.showMessageDialog(view, "Por favor, selecciona una salida para eliminar.", "Error", JOptionPane.ERROR_MESSAGE);
            return;
        }

        int confirmacion = JOptionPane.showConfirmDialog(view, "¿Estás seguro de que deseas eliminar esta salida?", "Confirmación", JOptionPane.YES_NO_OPTION);

        if (confirmacion == JOptionPane.YES_OPTION) {
            DefaultTableModel modeloTabla = (DefaultTableModel) view.tblSalidas.getModel();
            String amigo = (String) modeloTabla.getValueAt(filaSeleccionada, 0);
            String nombreSalida = (String) modeloTabla.getValueAt(filaSeleccionada, 1);
            String fecha = (String) modeloTabla.getValueAt(filaSeleccionada, 2);
            String lugar = (String) modeloTabla.getValueAt(filaSeleccionada, 3);

            boolean eliminado = model.eliminarSalida(amigo, nombreSalida, fecha, lugar);

            if (eliminado) {
                JOptionPane.showMessageDialog(view, "Salida eliminada con éxito.", "Éxito", JOptionPane.INFORMATION_MESSAGE);
                cargarSalidasEnTabla();
            } else {
                JOptionPane.showMessageDialog(view, "No se pudo eliminar la salida. Intenta nuevamente.", "Error", JOptionPane.ERROR_MESSAGE);
            }
        }
    }

    private void cargarSalidasEnTabla() {
        DefaultTableModel modeloTablaSalidas = (DefaultTableModel) view.tblSalidas.getModel();
        modeloTablaSalidas.setRowCount(0);

        List<Document> salidas = model.obtenerSalidas();
        for (Document salida : salidas) {
            modeloTablaSalidas.addRow(new Object[]{
                salida.getString("amigo"),
                salida.getString("nombreSalida"),
                salida.getString("fecha"),
                salida.getString("lugar")
            });
        }
    }
}

package Controller;

// Strategy - Interfaz
public interface FieldValidationStrategy {
    boolean validate(String value);
    String getErrorMessage();
}

// Estrategia concreta
class NameValidationStrategy implements FieldValidationStrategy {
    @Override
    public boolean validate(String value) {
        return value.matches("[a-zA-Z\\s]+");
    }

    @Override
    public String getErrorMessage() {
        return "El nombre solo puede contener letras.";
    }
}

// Estrategia concreta
class PhoneValidationStrategy implements FieldValidationStrategy {
    @Override
    public boolean validate(String value) {
        return value.matches("\\d{10}");
    }

    @Override
    public String getErrorMessage() {
        return "El número de teléfono debe tener 10 dígitos.";
    }
}

package Controller;

// Command - Interfaz
public interface ScheduleCommand {
    void execute();
}

package Controller;

import Model.MongoConnection;
import Model.modelDiary;
import Model.modelFace;
import Model.modelSchedule;
import Vista.DiaryProgram;
import Vista.FaceProgram;
import Vista.Resume;
import Vista.scheduleOutings;
import javax.swing.*;
import javax.swing.filechooser.FileNameExtensionFilter;
import javax.swing.table.DefaultTableModel;
import java.awt.Image;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.io.File;
import java.io.FileWriter;
import java.text.SimpleDateFormat;
import java.time.LocalDate;
import java.time.Period;
import java.time.ZoneId;
import java.util.Date;
import java.util.List;
import org.bson.Document;

public class controlDiary implements ActionListener {

    private modelDiary model;
    private DiaryProgram view;
    private MongoConnection conexion;
    private ImageIcon imagenSeleccionada;

    public controlDiary(modelDiary model, DiaryProgram view) {
        this.model = model;
        this.view = view;
        this.conexion = MongoConnection.getInstance();

        this.view.btnAgregar.addActionListener(this);
        this.view.btnActualizar.addActionListener(this);
        this.view.btnEliminar.addActionListener(this);
        this.view.btnBuscar.addActionListener(this);
        this.view.btnMostrarPerfil.addActionListener(this);
        this.view.btnCerrarSesion.addActionListener(this);
        this.view.btnCargarFoto.addActionListener(this);
        this.view.btnCargarDatos.addActionListener(this);
        this.view.btnAgendarSalida.addActionListener(this);
        this.view.btnLimpiarDatos.addActionListener(this);
        this.view.btnDescargarCSV.addActionListener(this);
        this.view.btnDescargarJSON.addActionListener(this);

        this.view.txtDatos.addMouseListener(new java.awt.event.MouseAdapter() {
            public void mouseClicked(java.awt.event.MouseEvent evt) {
                seleccionarFilaTabla();
            }
        });
        mostrarAmigosEnTabla();
    }

    @Override
    public void actionPerformed(ActionEvent e) {
        if (e.getSource() == view.btnAgregar) {
            new AgregarAmigoAction(model, view).execute();
        } else if (e.getSource() == view.btnActualizar) {
            new ActualizarAmigoAction(model, view).execute();
        } else if (e.getSource() == view.btnEliminar) {
            new EliminarAmigoAction(model, view).execute();
        } else if (e.getSource() == view.btnBuscar) {
            new BuscarAmigoAction(model, view).execute();
        } else if (e.getSource() == view.btnMostrarPerfil) {
            mostrarPerfil();
        } else if (e.getSource() == view.btnCerrarSesion) {
            cerrarSesion();
        } else if (e.getSource() == view.btnCargarFoto) {
            cargarImagen();
        } else if (e.getSource() == view.btnCargarDatos) {
            cargarDatosEnTabla();
        } else if (e.getSource() == view.btnAgendarSalida) {
            agendarSalidita();
        } else if (e.getSource() == view.btnLimpiarDatos) {
            limpiarCampos();
        } else if (e.getSource() == view.btnDescargarCSV) {
            exportarDatosCSV();
        } else if (e.getSource() == view.btnDescargarJSON) {
            exportarDatosJSON();
        }
    }

    private void cargarDatosEnTabla() {
        DefaultTableModel modeloTabla = (DefaultTableModel) view.txtDatos.getModel();
        modeloTabla.setRowCount(0);

        for (Document amigo : model.obtenerAmigos()) {
            modeloTabla.addRow(new Object[]{
                amigo.getString("genero"),
                amigo.getString("nombreApodo"),
                amigo.getString("fecha"),
                amigo.getString("numero"),
                amigo.getString("comida"),
                amigo.getInteger("edad"),
                amigo.getString("color"),
                amigo.getString("musica"),
                amigo.getString("importante")
            });
        }

        if (modeloTabla.getRowCount() == 0) {
            JOptionPane.showMessageDialog(view, "No hay datos para mostrar en la tabla.");
        } else {
            JOptionPane.showMessageDialog(view, "Datos cargados correctamente.");
            limpiarCampos();
        }
    }

    private void cargarImagen() {
        JFileChooser fileChooser = new JFileChooser();
        fileChooser.setDialogTitle("Seleccionar Imagen");
        fileChooser.setFileFilter(new FileNameExtensionFilter("Archivos de imagen", "jpg", "png", "gif"));

        int resultado = fileChooser.showOpenDialog(view);
        if (resultado == JFileChooser.APPROVE_OPTION) {
            File archivo = fileChooser.getSelectedFile();
            String pathImagen = archivo.getAbsolutePath();

            ImageIcon icono = new ImageIcon(pathImagen);
            Image imagen = icono.getImage().getScaledInstance(150, 150, Image.SCALE_DEFAULT);
            view.lblFoto.setIcon(new ImageIcon(imagen));

            imagenSeleccionada = icono;
        }
    }

    private void mostrarPerfil() {
        int filaSeleccionada = view.txtDatos.getSelectedRow();
        if (filaSeleccionada >= 0) {
            String nombreApodo = (String) view.txtDatos.getValueAt(filaSeleccionada, 1);
            List<Document> amigos = model.buscarAmigo(new Document("nombreApodo", nombreApodo));

            if (amigos.isEmpty()) {
                JOptionPane.showMessageDialog(view, "No se encontró el amigo con el nombre especificado.");
                return;
            }

            Document amigo = amigos.get(0);

            Resume resumeView = new Resume();
            resumeView.lblNombreUsuario.setText(amigo.getString("nombreApodo"));
            resumeView.lblNacimiento.setText(amigo.getString("fecha"));
            resumeView.lblNumeroTelefono.setText(amigo.getString("numero"));
            resumeView.lblComidaFavorita.setText(amigo.getString("comida"));

            String datosTexto = "Género: " + amigo.getString("genero") + "\n"
                    + "Edad: " + amigo.getInteger("edad") + "\n"
                    + "Color favorito: " + amigo.getString("color") + "\n"
                    + "Género musical: " + amigo.getString("musica") + "\n"
                    + "Importante: " + amigo.getString("importante");
            resumeView.txtDatosArea.setText(datosTexto);

            if (imagenSeleccionada != null) {
                resumeView.lblImagen.setIcon(imagenSeleccionada);
            }

            resumeView.setVisible(true);
        } else {
            JOptionPane.showMessageDialog(view, "Por favor, selecciona un amigo de la tabla para ver su perfil.");
        }
    }

    private void cerrarSesion() {
        int respuesta = JOptionPane.showConfirmDialog(view, "¿Estás seguro de que deseas cerrar sesión?", "Confirmar cierre de sesión", JOptionPane.YES_NO_OPTION);
        if (respuesta == JOptionPane.YES_OPTION) {
            view.dispose();

            FaceProgram vistaInicio = new FaceProgram();
            modelFace modeloInicio = new modelFace();
            controlFace controllerInicio = new controlFace(vistaInicio, modeloInicio);
            vistaInicio.setVisible(true);

            JOptionPane.showMessageDialog(vistaInicio, "Sesión cerrada.");
        }
    }

    private void limpiarCampos() {
        view.txtNombre.setText("");
        view.jDFecha.setDate(null);
        view.txtNumeroTelefono.setText("");
        view.txtComidaFavorita.setText("");
        view.spinEdad.setValue(0);
        view.cboxColor.setSelectedIndex(0);
        view.listMusica.clearSelection();
        view.txtImportante.setText("");
        view.buttonGroup1.clearSelection();
    }

    private void mostrarAmigosEnTabla() {
        DefaultTableModel modeloTabla = (DefaultTableModel) view.txtDatos.getModel();
        modeloTabla.setRowCount(0);

        for (Document amigo : model.obtenerAmigos()) {
            modeloTabla.addRow(new Object[]{
                amigo.getString("genero"),
                amigo.getString("nombreApodo"),
                amigo.getString("fecha"),
                amigo.getString("numero"),
                amigo.getString("comida"),
                amigo.getInteger("edad"),
                amigo.getString("color"),
                amigo.getString("musica"),
                amigo.getString("importante")
            });
        }
    }

    private void seleccionarFilaTabla() {
        int filaSeleccionada = view.txtDatos.getSelectedRow();
        if (filaSeleccionada >= 0) {
            String genero = (String) view.txtDatos.getValueAt(filaSeleccionada, 0);
            String nombreApodo = (String) view.txtDatos.getValueAt(filaSeleccionada, 1);
            Document amigo = model.buscarAmigo(new Document("nombreApodo", nombreApodo)).get(0);

            view.txtNombre.setText(amigo.getString("nombreApodo"));
            try {
                Date fecha = new SimpleDateFormat("dd/MM/yyyy").parse(amigo.getString("fecha"));
                view.jDFecha.setDate(fecha);
            } catch (java.text.ParseException e) {
                e.printStackTrace();
            }
            view.txtNumeroTelefono.setText(amigo.getString("numero"));
            view.txtComidaFavorita.setText(amigo.getString("comida"));
            view.spinEdad.setValue(amigo.getInteger("edad"));
            view.cboxColor.setSelectedItem(amigo.getString("color"));
            view.listMusica.setSelectedValue(amigo.getString("musica"), true);
            view.txtImportante.setText(amigo.getString("importante"));

            if ("Masculino".equals(genero)) {
                view.rdMasculino.setSelected(true);
            } else if ("Femenino".equals(genero)) {
                view.rdFemenino.setSelected(true);
            }
        }
    }

    private void agendarSalidita() {
        DefaultTableModel modeloTabla = (DefaultTableModel) view.txtDatos.getModel();
        int rowCount = modeloTabla.getRowCount();
        Object[][] datosAmigos = new Object[rowCount][modeloTabla.getColumnCount()];

        for (int i = 0; i < rowCount; i++) {
            for (int j = 0; j < modeloTabla.getColumnCount(); j++) {
                datosAmigos[i][j] = modeloTabla.getValueAt(i, j);
            }
        }

        scheduleOutings vista = new scheduleOutings();
        modelSchedule modelo = new modelSchedule();
        controlSchedule controller = new controlSchedule(vista, modelo);

        controller.setDatosAmigos(datosAmigos);

        vista.setVisible(true);
    }

    private void exportarDatosCSV() {
        JFileChooser fileChooser = new JFileChooser();
        fileChooser.setDialogTitle("Guardar archivo CSV");
        fileChooser.setFileFilter(new javax.swing.filechooser.FileNameExtensionFilter("Archivo CSV", "csv"));

        int seleccion = fileChooser.showSaveDialog(view);
        if (seleccion == JFileChooser.APPROVE_OPTION) {
            File archivo = fileChooser.getSelectedFile();
            if (!archivo.getName().endsWith(".csv")) {
                archivo = new File(archivo.getAbsolutePath() + ".csv");
            }

            try (FileWriter writer = new FileWriter(archivo)) {
                writer.write("Género,Nombre/Apodo,Fecha,Número,Comida,Edad,Color,Música,Importante\n");

                for (Document amigo : model.obtenerAmigos()) {
                    writer.write(String.format("%s,%s,%s,%s,%s,%d,%s,%s,%s\n",
                            amigo.getString("genero"),
                            amigo.getString("nombreApodo"),
                            amigo.getString("fecha"),
                            amigo.getString("numero"),
                            amigo.getString("comida"),
                            amigo.getInteger("edad"),
                            amigo.getString("color"),
                            amigo.getString("musica"),
                            amigo.getString("importante")));
                }

                JOptionPane.showMessageDialog(view, "Datos exportados correctamente a CSV.", "Éxito", JOptionPane.INFORMATION_MESSAGE);
            } catch (Exception ex) {
                JOptionPane.showMessageDialog(view, "Error al exportar datos a CSV: " + ex.getMessage(), "Error", JOptionPane.ERROR_MESSAGE);
            }
        }
    }

    private void exportarDatosJSON() {
        JFileChooser fileChooser = new JFileChooser();
        fileChooser.setDialogTitle("Guardar archivo JSON");
        fileChooser.setFileFilter(new javax.swing.filechooser.FileNameExtensionFilter("Archivo JSON", "json"));

        int seleccion = fileChooser.showSaveDialog(view);
        if (seleccion == JFileChooser.APPROVE_OPTION) {
            File archivo = fileChooser.getSelectedFile();
            if (!archivo.getName().endsWith(".json")) {
                archivo = new File(archivo.getAbsolutePath() + ".json");
            }

            try (FileWriter writer = new FileWriter(archivo)) {
                List<Document> amigos = model.obtenerAmigos();
                writer.write("[\n");

                for (int i = 0; i < amigos.size(); i++) {
                    writer.write(amigos.get(i).toJson());
                    if (i < amigos.size() - 1) {
                        writer.write(",\n");
                    }
                }

                writer.write("\n]");
                JOptionPane.showMessageDialog(view, "Datos exportados correctamente a JSON.", "Éxito", JOptionPane.INFORMATION_MESSAGE);
            } catch (Exception ex) {
                JOptionPane.showMessageDialog(view, "Error al exportar datos a JSON: " + ex.getMessage(), "Error", JOptionPane.ERROR_MESSAGE);
            }
        }
    }
}

package Controller;

import Model.MongoConnection;
import Model.modelDiary;
import Model.modelFace;
import Vista.FaceProgram;
import javax.swing.JOptionPane;
import Vista.DiaryProgram;
import com.mongodb.client.MongoCollection;
import org.bson.Document;

import java.io.IOException;
import java.util.regex.Pattern;

public class controlFace {

    private FaceProgram vista;
    private modelFace modelo;
    private static final String ERROR_USUARIO_OBLIGATORIO = "El usuario es obligatorio.";
    private static final String ERROR_CLAVE_OBLIGATORIA = "La clave es obligatoria.";

    public controlFace(FaceProgram vista, modelFace modelo) {
        this.vista = vista;
        this.modelo = modelo;
        this.vista.btnIngresar.addActionListener(e -> ingresar());
        this.vista.btnRegistrar.addActionListener(e -> registrarUsuario());
        this.vista.btnRecordar.addActionListener(e -> recordarUsuario());
        this.vista.btnRecuperar.addActionListener(e -> recuperarContraseña());
        this.vista.btnSalir.addActionListener(e -> salir());
    }

    private void ingresar() {
        limpiarErrores();
        String usuario = vista.txtUsuario.getText();
        String clave = vista.txtClave.getText();

        boolean hayErrores = false;

        if (usuario.isEmpty()) {
            vista.lblErrorUsuario.setText(ERROR_USUARIO_OBLIGATORIO);
            vista.lblErrorUsuario.setVisible(true);
            hayErrores = true;
        }
        if (clave.isEmpty()) {
            vista.lblErrorClave.setText(ERROR_CLAVE_OBLIGATORIA);
            vista.lblErrorClave.setVisible(true);
            hayErrores = true;
        }

        if (!hayErrores) {
            System.out.println("Intentando iniciar sesión con usuario: " + usuario + " y clave: " + clave);
            if (modelo.verificarUsuario(usuario, clave)) {
                JOptionPane.showMessageDialog(vista, "Inicio de sesión exitoso");

                vista.dispose();

                DiaryProgram view = new DiaryProgram();
                modelDiary model = new modelDiary();
                controlDiary controller = new controlDiary(model, view);
                view.setVisible(true);
            } else {
                JOptionPane.showMessageDialog(vista, "Usuario o contraseña incorrectos");
            }
        }
    }

    private void registrarUsuario() {
        limpiarErrores();

        String nombre = vista.txtNombre.getText().trim();
        String apellido = vista.txtApellido.getText().trim();
        String correo = vista.txtCorreoR.getText().trim();
        String usuario = vista.txtUsuarioR.getText().trim();
        String clave = vista.txtClaveR.getText().trim();

        boolean hayErrores = false;

        if (nombre.isEmpty()) {
            vista.lblErrorNombre.setText("El nombre es obligatorio.");
            vista.lblErrorNombre.setVisible(true);
            hayErrores = true;
        } else if (!nombre.matches("^[a-zA-ZáéíóúÁÉÍÓÚñÑ ]+$")) {
            vista.lblErrorNombre.setText("El nombre solo puede contener letras.");
            vista.lblErrorNombre.setVisible(true);
            hayErrores = true;
        } else {
            vista.lblErrorNombre.setText(""); // Limpia el mensaje de error si es válido
        }

        if (apellido.isEmpty()) {
            vista.lblErrorApellido.setText("El apellido es obligatorio.");
            vista.lblErrorApellido.setVisible(true);
            hayErrores = true;
        } else if (!apellido.matches("^[a-zA-ZáéíóúÁÉÍÓÚñÑ ]+$")) {
            vista.lblErrorApellido.setText("El apellido solo puede contener letras.");
            vista.lblErrorApellido.setVisible(true);
            hayErrores = true;
        } else {
            vista.lblErrorApellido.setText(""); // Limpia el mensaje de error si es válido
        }

        if (correo.isEmpty()) {
            vista.lblErrorCorreo.setText("El correo es obligatorio.");
            vista.lblErrorCorreo.setVisible(true);
            hayErrores = true;
        } else if (!isEmailValid(correo)) {
            vista.lblErrorCorreo.setText("El formato del correo es inválido.");
            vista.lblErrorCorreo.setVisible(true);
            hayErrores = true;
        } else {
            vista.lblErrorCorreo.setText(""); // Limpia el mensaje de error si es válido
        }

        if (usuario.isEmpty()) {
            vista.lblErrorUsuarioRegistro.setText("El usuario es obligatorio.");
            vista.lblErrorUsuarioRegistro.setVisible(true);
            hayErrores = true;
        } else if (!usuario.matches(".*[a-zA-Z].*") || usuario.matches("^[0-9]+$")) {
            vista.lblErrorUsuarioRegistro.setText("El usuario debe contener al menos una letra y no solo números.");
            vista.lblErrorUsuarioRegistro.setVisible(true);
            hayErrores = true;
        } else {
            vista.lblErrorUsuarioRegistro.setText(""); // Limpia el mensaje de error si es válido
        }

        if (clave.isEmpty()) {
            vista.lblErorClaveRegistro.setText("La clave es obligatoria.");
            vista.lblErorClaveRegistro.setVisible(true);
            hayErrores = true;
        } else {
            vista.lblErorClaveRegistro.setText(""); // Limpia el mensaje de error si es válido
        }

        if (!hayErrores) {
            if (modelo.registrarUsuario(nombre, apellido, correo, usuario, clave)) {
                JOptionPane.showMessageDialog(vista, "Registro exitoso");
            } else {
                JOptionPane.showMessageDialog(vista, "Error al registrar usuario");
            }
        }
    }

private void recordarUsuario() {
        limpiarErrores();

        String correoIngresado = JOptionPane.showInputDialog(vista, "Ingresa tu correo para recordar el usuario:");

        if (correoIngresado != null && !correoIngresado.isEmpty()) {
            MongoCollection<Document> collection = MongoConnection.getInstance().getCollection();

            Document usuarioDoc = collection.find(new Document("correo", correoIngresado)).first();

            if (usuarioDoc != null) {
                String usuario = usuarioDoc.getString("usuario");
                String clave = usuarioDoc.getString("clave");

                vista.txtUsuario.setText(usuario);
                vista.txtClave.setText(clave);

                JOptionPane.showMessageDialog(vista, "Usuario encontrado. Los campos han sido completados.", "Confirmación", JOptionPane.INFORMATION_MESSAGE);
            } else {
                JOptionPane.showMessageDialog(vista, "No se encontró un usuario con ese correo.", "Error", JOptionPane.ERROR_MESSAGE);
            }
        } else {
            JOptionPane.showMessageDialog(vista, "Por favor, ingresa un correo.", "Error", JOptionPane.ERROR_MESSAGE);
        }
    }

    private void recuperarContraseña() {
        limpiarErrores();

        String usuarioGuardado = vista.txtUsuario.getText().trim();

        if (usuarioGuardado != null && !usuarioGuardado.isEmpty()) {
            MongoCollection<Document> collection = MongoConnection.getInstance().getCollection();

            Document usuarioDoc = collection.find(new Document("usuario", usuarioGuardado)).first();

            if (usuarioDoc != null) {
                String clave = usuarioDoc.getString("clave");

                vista.txtClave.setText(clave);

                JOptionPane.showMessageDialog(vista, "Usuario encontrado. Los campos han sido completados.", "Confirmación", JOptionPane.INFORMATION_MESSAGE);
            } else {
                JOptionPane.showMessageDialog(vista, "Usuario no encontrado.", "Error", JOptionPane.ERROR_MESSAGE);
            }
        } else {
            JOptionPane.showMessageDialog(vista, "Por favor, ingresa un nombre de usuario.", "Error", JOptionPane.ERROR_MESSAGE);
        }
    }

    private void limpiarErrores() {
        vista.lblErrorUsuario.setVisible(false);
        vista.lblErrorClave.setVisible(false);
        vista.lblErrorApellido.setVisible(false);
        vista.lblErrorNombre.setVisible(false);
        vista.lblErrorCorreo.setVisible(false);
        vista.lblErrorUsuarioRegistro.setVisible(false);
        vista.lblErorClaveRegistro.setVisible(false);
    }

    private boolean isEmailValid(String email) {
        String emailRegex = "^[a-zA-Z0-9_+&*-]+(?:\\.[a-zA-Z0-9_+&*-]+)*@(?:[a-zA-Z0-9-]+\\.)+[a-zA-Z]{2,7}$";
        return Pattern.compile(emailRegex).matcher(email).matches();
    }

    private void salir() {
        int respuesta = JOptionPane.showConfirmDialog(vista, "¿Estás seguro de que deseas salir?", "Confirmar salida", JOptionPane.YES_NO_OPTION);
        if (respuesta == JOptionPane.YES_OPTION) {
            vista.dispose();
            System.exit(0);
        }
    }
}

package Controller;

import Model.modelSchedule;
import Vista.scheduleOutings;
import javax.swing.JOptionPane;
import javax.swing.table.DefaultTableModel;
import org.bson.Document;
import java.util.List;
import java.util.Date;
import java.text.SimpleDateFormat;
import java.time.LocalDate;
import java.time.ZoneId;

public class controlSchedule {

    private scheduleOutings vista;
    private modelSchedule modelo;

    public controlSchedule(scheduleOutings vista, modelSchedule modelo) {
        this.vista = vista;
        this.modelo = modelo;
        this.vista.btnAgendarSalida.addActionListener(e -> new AgendarSalidaCommand(modelo, vista).execute());
        this.vista.btnBuscarSalida.addActionListener(e -> new BuscarSalidaCommand(modelo, vista).execute());
        this.vista.btnRegresar.addActionListener(e -> regresar());
        this.vista.btnBorrar.addActionListener(e -> new EliminarSalidaCommand(modelo, vista).execute());

        cargarAmigosEnTabla();
        cargarSalidasEnTabla();
    }

    private Object[][] datosAmigos;

    public void setDatosAmigos(Object[][] datos) {
        this.datosAmigos = datos;
        cargarAmigosEnTabla();
    }

    private void cargarAmigosEnTabla() {
        DefaultTableModel modeloTabla = (DefaultTableModel) vista.tblDatosSalidas.getModel();
        modeloTabla.setRowCount(0);

        if (datosAmigos != null && datosAmigos.length > 0) {
            for (Object[] amigo : datosAmigos) {
                modeloTabla.addRow(new Object[]{
                    amigo[0], 
                    amigo[1], 
                    amigo[2], 
                    amigo[3], 
                    amigo[4], 
                    amigo[5], 
                    amigo[6], 
                    amigo[7], 
                    amigo[8] 
                });
            }
        } else {
            List<String> amigos = modelo.obtenerAmigos();
            if (amigos.isEmpty()) {
                JOptionPane.showMessageDialog(vista, "No hay amigos para mostrar.");
            } else {
                for (String amigo : amigos) {
                    modeloTabla.addRow(new Object[]{amigo});
                }
            }
        }
    }

    private void cargarSalidasEnTabla() {
        DefaultTableModel modeloTablaSalidas = (DefaultTableModel) vista.tblSalidas.getModel();
        modeloTablaSalidas.setRowCount(0);

        List<Document> salidas = modelo.obtenerSalidas();
        for (Document salida : salidas) {
            modeloTablaSalidas.addRow(new Object[]{
                salida.getString("amigo"),
                salida.getString("nombreSalida"),
                salida.getString("fecha"),
                salida.getString("lugar")
            });
        }
    }

    private void regresar() {
        vista.dispose();
    }
}
