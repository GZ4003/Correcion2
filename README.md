#!/bin/bash

# Función para registrar eventos en syslog
log_event() {
    logger -t user_admin "$1"
}

# Crear grupos
for group_name in Desarrollo Operaciones Ingeniería; do
    if ! getent group "$group_name" > /dev/null 2>&1; then
        sudo groupadd "$group_name"
        log_event "Grupo creado: $group_name"
    else
        log_event "El grupo $group_name ya existe"
    fi
done

# Datos de usuarios
users_data=(
    "usuario1 Desarrollo contraseña1"
    "usuario2 Operaciones contraseña2"
    "usuario3 Ingeniería contraseña3"
    "usuario4 Desarrollo contraseña4"
    "usuario5 Operaciones contraseña5"
    "usuario6 Ingeniería contraseña6"
)

# Crear usuarios y asignarlos a grupos
for user_info in "${users_data[@]}"; do
    username=$(echo "$user_info" | cut -d ' ' -f 1)
    groupname=$(echo "$user_info" | cut -d ' ' -f 2)
    password=$(echo "$user_info" | cut -d ' ' -f 3)

    if ! id "$username" > /dev/null 2>&1; then
        sudo useradd -m -g "$groupname" -s /bin/bash "$username"
        echo "$username:$password" | sudo chpasswd
        log_event "Usuario creado: $username en el grupo $groupname"
    else
        log_event "El usuario $username ya existe"
    fi
done

# Crear carpetas y configurar ownership y permisos
for user_info in "${users_data[@]}"; do
    username=$(echo "$user_info" | cut -d ' ' -f 1)
    home_dir="/home/$username"

    # Crear carpeta si no existe
    if [ ! -d "$home_dir" ]; then
        sudo mkdir "$home_dir"
        log_event "Carpeta creada para: $username"
    fi

    # Cambiar ownership de la carpeta al usuario
    sudo chown "$username:$username" "$home_dir"
    log_event "Ownership cambiado a: $username para la carpeta: $home_dir"

    # Configurar permisos (lectura, escritura y ejecución para propietario y grupo)
    sudo chmod 770 "$home_dir"
    log_event "Permisos configurados para: $username en la carpeta: $home_dir"
done

echo "Script finalizado."
