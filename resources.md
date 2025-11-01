# ComfyUI Resources

## Essential Models

### Base Models
- **SD 1.5**: `v1-5-pruned-emaonly.ckpt` - Standard model, good for general use
- **SD 1.5 Inpainting**: For editing specific areas
- **SDXL 1.0**: Higher resolution, better quality but slower
- **SDXL Turbo**: Faster SDXL variant

### Recommended Checkpoints for Creatures/Animals
- **DreamShaper**: Good for fantasy creatures
- **Anything V5**: Anime/cartoon style
- **RealisticVision**: For realistic renders
- **Deliberate**: Versatile artistic model

## LoRA Models for Chicken/Creatures

### Animal LoRAs
- Cartoon animals
- Fantasy creatures
- Feather details enhancer
- Eye detail improver

### Style LoRAs
- Pixar style
- Studio Ghibli style
- Watercolor effect
- Low poly 3D

## Essential Custom Nodes

### Must-Have Node Packs
1. **ComfyUI Manager** - Install and manage other nodes
2. **Efficiency Nodes** - Workflow optimization
3. **Impact Pack** - Advanced selection and processing
4. **ControlNet Preprocessors** - For pose control
5. **AnimateDiff** - For animations
6. **Ultimate SD Upscale** - High-quality upscaling

### Utility Nodes
- **Image Saver Plus** - Better save options
- **Workflow Component** - Modular workflows
- **Primitives** - Basic value nodes
- **Math Expression** - Mathematical operations

## Learning Resources

### Documentation
- [Official ComfyUI GitHub](https://github.com/comfyanonymous/ComfyUI)
- [ComfyUI Examples](https://comfyanonymous.github.io/ComfyUI_examples/)
- [ComfyUI Wiki](https://github.com/comfyanonymous/ComfyUI/wiki)

### Video Tutorials
- ComfyUI Basic Tutorial Series
- Advanced Workflow Techniques
- Custom Node Development

### Community
- [ComfyUI Reddit](https://reddit.com/r/comfyui)
- [Discord Server](https://discord.gg/comfyui)
- [CivitAI](https://civitai.com) - Models and LoRAs

## Optimization Tips

### VRAM Management
- **6GB VRAM**: Use SD 1.5, batch size 1, lower resolution
- **8GB VRAM**: Can handle SDXL with optimizations
- **12GB+ VRAM**: Full SDXL, larger batches, higher resolutions

### Speed Optimization
1. Use `--lowvram` or `--medvram` flags if needed
2. Enable xformers: `--xformers`
3. Use faster samplers (DPM++ 2M, UniPC)
4. Reduce steps where possible
5. Use TAESD for preview (faster VAE)

### Quality Settings
- **Draft**: 10-15 steps, CFG 5-7, euler_a
- **Normal**: 20-25 steps, CFG 7-8, DPM++ 2M
- **High**: 30-40 steps, CFG 7-9, DPM++ 2M Karras
- **Maximum**: 50+ steps, CFG 7-8, DPM++ 3M SDE

## Workflow Templates

### Basic Text2Image
- Load Checkpoint → CLIP Encode → KSampler → VAE Decode → Save

### Image2Image
- Load Image → VAE Encode → KSampler → VAE Decode → Save

### Inpainting
- Load Image → Load Mask → VAE Encode → KSampler → VAE Decode → Save

### ControlNet
- Load Image → Preprocessor → Apply ControlNet → KSampler → Save

### Upscaling
- Load Image → Upscale Model → VAE Encode → KSampler → VAE Decode → Save

## Troubleshooting

### Common Issues
1. **Out of memory**: Reduce batch size, use --lowvram
2. **Slow generation**: Check sampler, reduce steps
3. **Poor quality**: Increase steps, adjust CFG, try different model
4. **Black images**: Check VAE, ensure proper connections
5. **Error loading checkpoint**: Verify file path and compatibility

### Performance Monitoring
- GPU usage: `nvidia-smi` or `rocm-smi`
- VRAM usage: Check in ComfyUI interface
- Generation time: Enable timing in settings

## Project-Specific Resources

### Chicken Creature References
- Real chicken anatomy guides
- Fantasy creature design principles
- Color theory for feathers
- Animation reference sheets

### Useful Prompting Guides
- Prompt weighting syntax
- Style mixing techniques
- Negative prompt best practices
- Seed manipulation strategies