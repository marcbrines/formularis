# formularis
## formA.php
<?php
// forms/formA.php

// Inicializar valores y errores
$errors = [];
$values = [
    'nombre' => '', 'apellido' => '', 'edad' => '',
    'direccion' => '', 'telefono' => '', 'email' => '', 'genero' => ''
];
$languages = [];

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    // Recoger datos y validar
    foreach ($values as $k => $v) {
        $values[$k] = isset($_POST[$k]) ? trim($_POST[$k]) : '';
        if ($k !== 'genero' && $values[$k] === '') {
            $errors[$k] = "El campo " . ucfirst($k) . " es obligatorio.";
        }
    }

    // Idiomas
    $languages = isset($_POST['idiomas']) ? $_POST['idiomas'] : [];

    // Validar formato de email
    if (!empty($values['email']) && !filter_var($values['email'], FILTER_VALIDATE_EMAIL)) {
        $errors['email_format'] = "Formato de email inválido.";
    }

    // Si no hay errores, mostrar datos
    if (empty($errors)) {
        function safe($s){ return htmlspecialchars($s, ENT_QUOTES, 'UTF-8'); }
        echo "<!doctype html><html><head><meta charset='utf-8'><title>Resultado Form A</title></head><body>";
        echo "<h1>Datos recibidos</h1>";
        echo "<p>Nombre: " . safe($values['nombre']) . " " . safe($values['apellido']) . "</p>";
        echo "<p>Edad: " . safe($values['edad']) . "</p>";
        echo "<p>Dirección: " . safe($values['direccion']) . "</p>";
        echo "<p>Teléfono: " . safe($values['telefono']) . "</p>";
        echo "<p>Email: " . safe($values['email']) . "</p>";
        echo "<p>Género: " . safe($values['genero']) . "</p>";
        echo "<p>Idiomas: " . (empty($languages) ? 'No seleccionado' : safe(implode(', ', $languages))) . "</p>";

        // Mensaje de bienvenida por idioma
        $mensajes = [
            'español' => 'Bienvenido/a',
            'inglés' => 'Welcome',
            'francés' => 'Bienvenue',
            'alemán' => 'Willkommen',
            'italiano' => 'Benvenuto/a'
        ];

        if (!empty($languages)) {
            echo "<h3>Saludos:</h3><ul>";
            foreach ($languages as $lang) {
                if (isset($mensajes[$lang])) {
                    echo "<li>" . $mensajes[$lang] . " (" . safe($lang) . ")</li>";
                } else {
                    echo "<li>Hola (" . safe($lang) . ")</li>";
                }
            }
            echo "</ul>";
        }

        echo "<p><a href='formA.php'>Volver al formulario</a></p>";
        echo "</body></html>";
        exit;
    }
}
?>
<!doctype html>
<html>
<head><meta charset="utf-8"><title>Formulario A — Datos personales</title></head>
<body>
  <h1>Formulario A — Datos personales</h1>

  <?php if (!empty($errors)): ?>
    <div style="color: red;">
      <ul>
        <?php foreach ($errors as $err) echo "<li>" . htmlspecialchars($err) . "</li>"; ?>
      </ul>
    </div>
  <?php endif; ?>

  <form method="post" action="formA.php">
    Nombre: <input type="text" name="nombre" value="<?=htmlspecialchars($values['nombre'])?>"><br><br>
    Apellido: <input type="text" name="apellido" value="<?=htmlspecialchars($values['apellido'])?>"><br><br>
    Edad: <input type="number" name="edad" value="<?=htmlspecialchars($values['edad'])?>"><br><br>
    Dirección: <input type="text" name="direccion" value="<?=htmlspecialchars($values['direccion'])?>"><br><br>
    Teléfono: <input type="text" name="telefono" value="<?=htmlspecialchars($values['telefono'])?>"><br><br>
    Email: <input type="email" name="email" value="<?=htmlspecialchars($values['email'])?>"><br><br>

    Género:<br>
    <label><input type="radio" name="genero" value="Masculino" <?= $values['genero']=='Masculino' ? 'checked' : '' ?>> Masculino</label>
    <label><input type="radio" name="genero" value="Femenino" <?= $values['genero']=='Femenino' ? 'checked' : '' ?>> Femenino</label>
    <label><input type="radio" name="genero" value="Otro" <?= $values['genero']=='Otro' ? 'checked' : '' ?>> Otro</label>
    <br><br>

    Idiomas:<br>
    <?php
      $opts = ['español','inglés','francés','alemán','italiano'];
      foreach ($opts as $o) {
        $checked = in_array($o, $languages) ? 'checked' : '';
        echo "<label><input type='checkbox' name='idiomas[]' value='$o' $checked> $o</label> ";
      }
    ?>
    <br><br>
    <button type="submit">Enviar</button>
  </form>
</body>
</html>


## formA_post.html

<!-- forms/formA_post.html -->
<!doctype html>
<html>
<head><meta charset="utf-8"><title>Form A - POST</title></head>
<body>
  <h1>Formulario A (envío a otra página)</h1>
  <form method="post" action="processar.php">
    Nombre: <input type="text" name="nombre"><br><br>
    Apellido: <input type="text" name="apellido"><br><br>
    Edad: <input type="number" name="edad"><br><br>
    Dirección: <input type="text" name="direccion"><br><br>
    Teléfono: <input type="text" name="telefono"><br><br>
    Email: <input type="email" name="email"><br><br>

    Género:<br>
    <label><input type="radio" name="genero" value="Masculino"> Masculino</label>
    <label><input type="radio" name="genero" value="Femenino"> Femenino</label>
    <label><input type="radio" name="genero" value="Otro"> Otro</label>
    <br><br>

    Idiomas:<br>
    <label><input type="checkbox" name="idiomas[]" value="español"> español</label>
    <label><input type="checkbox" name="idiomas[]" value="inglés"> inglés</label>
    <label><input type="checkbox" name="idiomas[]" value="francés"> francés</label>
    <br><br>

    <button type="submit">Enviar (POST)</button>
  </form>
</body>
</html>



## processar.php

<?php
// forms/processar.php
function safe($s){ return htmlspecialchars($s ?? '', ENT_QUOTES, 'UTF-8'); }
$errors = [];
$required = ['nombre','apellido','edad','direccion','telefono','email'];

if ($_SERVER['REQUEST_METHOD'] !== 'POST') {
    echo "<p>Acceso inválido. Usa <a href='formA_post.html'>formA_post.html</a></p>";
    exit;
}

// Validaciones básicas
foreach ($required as $r) {
    if (empty($_POST[$r])) $errors[] = "El campo " . ucfirst($r) . " es obligatorio.";
}

if (!empty($_POST['email']) && !filter_var($_POST['email'], FILTER_VALIDATE_EMAIL)) {
    $errors[] = "Formato de email inválido.";
}

if (!empty($errors)) {
    echo "<h2>Errores</h2><ul>";
    foreach ($errors as $e) echo "<li>" . safe($e) . "</li>";
    echo "</ul><p><a href='formA_post.html'>Volver</a></p>";
    exit;
}

// Mostrar los datos
echo "<!doctype html><html><head><meta charset='utf-8'><title>Processar</title></head><body>";
echo "<h1>Datos procesados (processar.php)</h1>";
foreach ($required as $r) {
    echo "<p>" . ucfirst($r) . ": " . safe($_POST[$r]) . "</p>";
}
if (!empty($_POST['genero'])) echo "<p>Género: " . safe($_POST['genero']) . "</p>";
if (!empty($_POST['idiomas'])) echo "<p>Idiomas: " . safe(implode(', ', $_POST['idiomas'])) . "</p>";
else echo "<p>No se seleccionó ningún idioma.</p>";

echo "<p><a href='formA_post.html'>Volver al formulario</a></p>";
echo "</body></html>";


## formB.html

<!-- forms/formB.html -->
<!doctype html>
<html>
<head><meta charset="utf-8"><title>Form B - subir .txt</title></head>
<body>
  <h1>Subir fichero de texto (.txt)</h1>
  <form method="post" action="upload.php" enctype="multipart/form-data">
    Seleccione archivo (.txt): <input type="file" name="archivo" accept=".txt,text/plain"><br><br>
    <button type="submit">Subir</button>
  </form>

  <p><a href="formA.php">Volver al Form A</a></p>
</body>
</html>


## upload.php

<?php
// forms/upload.php
$uploadDir = __DIR__ . '/../uploads/';
if (!is_dir($uploadDir)) {
    mkdir($uploadDir, 0755, true);
}

function safe($s){ return htmlspecialchars($s ?? '', ENT_QUOTES, 'UTF-8'); }

if ($_SERVER['REQUEST_METHOD'] !== 'POST') {
    echo "<p>Acceso inválido. Usa <a href='formB.html'>formB.html</a></p>";
    exit;
}

if (!isset($_FILES['archivo'])) {
    echo "<p>No se ha enviado ningún archivo. <a href='formB.html'>Volver</a></p>";
    exit;
}

$f = $_FILES['archivo'];

if ($f['error'] !== UPLOAD_ERR_OK) {
    echo "<p>Error en la subida: " . $f['error'] . "</p>";
    exit;
}

// Validar extensión y MIME
$ext = strtolower(pathinfo($f['name'], PATHINFO_EXTENSION));
if ($ext !== 'txt') {
    echo "<p>Extensión no permitida. Solo .txt</p>";
    exit;
}

$finfo = finfo_open(FILEINFO_MIME_TYPE);
$mime = finfo_file($finfo, $f['tmp_name']);
finfo_close($finfo);
if ($mime !== 'text/plain') {
    echo "<p>Tipo MIME no permitido: " . safe($mime) . "</p>";
    exit;
}

// Guardar con nombre único
$base = pathinfo($f['name'], PATHINFO_FILENAME);
$base = preg_replace('/[^A-Za-z0-9_\-]/', '_', $base);
$target = $uploadDir . $base . '_' . time() . '.txt';

if (!move_uploaded_file($f['tmp_name'], $target)) {
    echo "<p>No se pudo mover el fichero al directorio de destino.</p>";
    exit;
}

// Mostrar contenido del archivo
$content = file_get_contents($target);
echo "<!doctype html><html><head><meta charset='utf-8'><title>Upload result</title></head><body>";
echo "<h1>Fichero subido correctamente</h1>";
echo "<p>Guardado como: " . safe(basename($target)) . "</p>";
echo "<h3>Contenido del archivo:</h3><pre>" . safe($content) . "</pre>";
echo "<p><a href='formB.html'>Subir otro archivo</a></p>";
echo "</body></html>";
