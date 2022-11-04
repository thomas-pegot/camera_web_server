# Warning! This code was automatically generated for projects
# without default 'CMakeLists.txt' file.

set(COMPONENT_SRCS  "app_main.c" "app_wifi.c" "app_camera.c" "app_httpd.c" 
                "app_mdns.c" "image_util.c")

set(COMPONENT_ADD_INCLUDEDIRS "../include")

set(COMPONENT_EMBED_FILES
        "../www/index_ov2640.html.gz"
        "../www/index_ov3660.html.gz"
        "../www/index_ov5640.html.gz"
        "../www/monitor.html.gz")

set(COMPONENT_REQUIRES
        esp32-motion
        esp32-camera
        nvs_flash
        esp_http_server
        fb_gfx
        mdns
        )


register_component()
