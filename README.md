# azi03 3hffjylNK>HIRjh?
.KH>GUhhg.u
# Blender Python Script: animate_nft_1920frames.py     jhgjgjh
# This script creates a parallax-style 3D animation from a flat image for NFT purposes.
# Run inside Blender's Text Editor -> Run Script.

import bpy  
import math
import os

# ---------------- SETTINGS ----------------
img_path = "/mnt/data/arezo.msp1 - Copy.png"       # Path to your source image
output_path = "/tmp/nft_animation_1920frames.mp4"  # Output video file path 

duration_seconds = 10
total_frames = 1920
fps = total_frames // duration_seconds  # 192 fps

scene = bpy.context.scene
scene.render.engine = 'BLENDER_EEVEE'
scene.render.image_settings.file_format = 'FFMPEG'
scene.render.ffmpeg.format = 'MPEG4'
scene.render.ffmpeg.codec = 'H264'
scene.render.ffmpeg.constant_rate_factor = 'PERCENTAGE'
scene.render.resolution_x = 1920
scene.render.resolution_y = 1080
scene.render.resolution_percentage = 100
scene.frame_start = 1
scene.frame_end = total_frames
scene.render.fps = fps
scene.render.filepath = output_path

# ---------------- CLEAR SCENE ----------------
bpy.ops.object.select_all(action='SELECT')
bpy.ops.object.delete(use_global=False)

# ---------------- CREATE CAMERA ----------------
cam_data = bpy.data.cameras.new("Cam")
cam = bpy.data.objects.new("Camera", cam_data)
scene.collection.objects.link(cam)
scene.camera = cam

# Camera positions and rotations for start/mid/end (loop animation)
cam_start_loc = (0.0, -3.6, 0.55)
cam_mid_loc   = (0.15, -3.0, 0.60)
cam_start_rot = (math.radians(78), 0, 0)
cam_mid_rot   = (math.radians(79), 0, math.radians(3.5))

cam.location = cam_start_loc
cam.rotation_euler = cam_start_rot

# ---------------- CREATE LIGHT ----------------
light_data = bpy.data.lights.new(name="KeyLight", type='AREA')
light = bpy.data.objects.new(name="KeyLight", object_data=light_data)
scene.collection.objects.link(light)
light.location = (2.0, -2.0, 3.0)
light.data.energy = 1500

# ---------------- FUNCTION: CREATE TEXTURED PLANE ----------------
def make_textured_plane(name, z_offset, scale=1.0, alpha=1.0):
    """Creates a plane with the given image texture, positioned at z_offset."""
    bpy.ops.mesh.primitive_plane_add(size=2.0, location=(0, z_offset, 0))
    obj = bpy.context.active_object
    obj.scale = (scale, scale, scale)
    obj.name = name

    mat = bpy.data.materials.new(name + "_MAT")
    mat.use_nodes = True
    nodes = mat.node_tree.nodes
    links = mat.node_tree.links
    nodes.clear()

    output = nodes.new(type="ShaderNodeOutputMaterial")
    principled = nodes.new(type="ShaderNodeBsdfPrincipled")
    tex = nodes.new(type="ShaderNodeTexImage")

    try:
        tex.image = bpy.data.images.load(img_path)
    except Exception as e:
        raise RuntimeError(f"Cannot load image at path: {img_path}\n{e}")

    tex.interpolation = 'Smart'
    links.new(tex.outputs['Color'], principled.inputs['Base Color'])

    # If the image has alpha channel
    if 'Alpha' in tex.outputs:
        links.new(tex.outputs['Alpha'], principled.inputs['Alpha'])
        mat.blend_method = 'BLEND'
    else:
        mat.blend_method = 'OPAQUE'

    links.new(principled.outputs['BSDF'], output.inputs['Surface'])
    obj.data.materials.append(mat)
    return obj

# ---------------- CREATE PLANES FOR PARALLAX EFFECT ----------------
back  = make_textured_plane("Back",  z_offset=0.18, scale=1.02, alpha=0.85)
mid   = make_textured_plane("Mid",   z_offset=0.09, scale=1.005, alpha=0.95)
front = make_textured_plane("Front", z_offset=0.00, scale=1.0, alpha=1.0)

# Slight rotations for better depth illusion
back.rotation_euler = (0, 0, math.radians(0.8))
mid.rotation_euler  = (0, 0, math.radians(0.4))
front.rotation_euler= (0, 0, 0)

# ---------------- CAMERA ANIMATION ----------------
scene.frame_set(1)
cam.location = cam_start_loc
cam.rotation_euler = cam_start_rot
cam.keyframe_insert(data_path='location', frame=1)
cam.keyframe_insert(data_path='rotation_euler', frame=1)

mid_frame = total_frames // 2
scene.frame_set(mid_frame)
cam.location = cam_mid_loc
cam.rotation_euler = cam_mid_rot
cam.keyframe_insert(data_path='location', frame=mid_frame)
cam.keyframe_insert(data_path='rotation_euler', frame=mid_frame)

scene.frame_set(total_frames)
cam.location = cam_start_loc
cam.rotation_euler = cam_start_rot
cam.keyframe_insert(data_path='location', frame=total_frames)
cam.keyframe_insert(data_path='rotation_euler', frame=total_frames)

# ---------------- LAYER PARALLAX ANIMATION ----------------
for i, obj in enumerate([back, mid, front]):
    scene.frame_set(1)
    obj.keyframe_insert(data_path='location', frame=1)
    scene.frame_set(mid_frame)
    obj.location.x += 0.03 * (i - 1)
    obj.keyframe_insert(data_path='location', frame=mid_frame)
    scene.frame_set(total_frames)
    obj.location.x -= 0.0
    obj.keyframe_insert(data_path='location', frame=total_frames)

# ---------------- SET LINEAR INTERPOLATION ----------------
def set_linear_interpolation(obj):
    """Ensures smooth, consistent motion (linear interpolation)."""
    if obj.animation_data and obj.animation_data.action:
        for fcurve in obj.animation_data.action.fcurves:
            for kp in fcurve.keyframe_points:
                kp.interpolation = 'LINEAR'

set_linear_interpolation(cam)
for o in (back, mid, front):
    set_linear_interpolation(o)

     # ---------------- RENDER ANIMATION ----------------
scene.render.filepath = output_path
bpy.ops.render.render(animation=True)
print(f"Rendered to: {output_path}")



   
