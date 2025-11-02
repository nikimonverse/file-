<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>AI File Converter</title>
  <style>
    * {
      box-sizing: border-box;
      font-family: "Poppins", sans-serif;
    }
    body {
      background: radial-gradient(circle at top, #0f0c29, #302b63, #24243e);
      color: #fff;
      height: 100vh;
      display: flex;
      justify-content: center;
      align-items: center;
    }
    .converter-box {
      background: rgba(255, 255, 255, 0.06);
      backdrop-filter: blur(12px);
      border: 1px solid rgba(255, 255, 255, 0.1);
      border-radius: 18px;
      width: 400px;
      padding: 35px;
      text-align: center;
      box-shadow: 0 0 25px rgba(138, 43, 226, 0.3);
      animation: fadeIn 1s ease-in-out;
    }
    h1 {
      color: #b388ff;
      font-size: 1.6rem;
      margin-bottom: 10px;
    }
    p {
      color: #bbb;
      font-size: 0.9rem;
      margin-bottom: 25px;
    }

    .drop-zone {
      border: 2px dashed rgba(255, 255, 255, 0.3);
      border-radius: 12px;
      padding: 35px;
      cursor: pointer;
      transition: all 0.3s ease;
      color: #aaa;
    }
    .drop-zone.dragover {
      background: rgba(138, 43, 226, 0.2);
      border-color: #9b5de5;
      color: #fff;
      box-shadow: 0 0 15px rgba(138, 43, 226, 0.4);
    }

    select, button {
      width: 100%;
      margin-top: 15px;
      padding: 12px;
      border-radius: 10px;
      border: none;
      outline: none;
      font-size: 0.95rem;
    }

    select {
      background: rgba(255, 255, 255, 0.1);
      color: #b388ff;
    }

    button {
      background: linear-gradient(90deg, #6c63ff, #8e2de2);
      color: #fff;
      font-weight: 600;
      cursor: pointer;
      transition: 0.3s;
    }
    button:hover {
      background: linear-gradient(90deg, #8e2de2, #4a00e0);
      transform: scale(1.03);
    }

    img {
      margin-top: 20px;
      max-width: 100%;
      border-radius: 10px;
      border: 1px solid rgba(255,255,255,0.2);
      box-shadow: 0 0 15px rgba(138,43,226,0.3);
    }

    /* Stylish Download Button */
    .download-btn {
      display: inline-block;
      margin-top: 15px;
      margin-right: 10px;
      padding: 10px 18px;
      border-radius: 8px;
      border: none;
      background: linear-gradient(90deg, #00c6ff, #0072ff);
      color: #fff;
      font-weight: 600;
      text-decoration: none;
      transition: all 0.3s ease;
      box-shadow: 0 0 10px rgba(0, 114, 255, 0.4);
    }
    .download-btn:hover {
      background: linear-gradient(90deg, #0072ff, #00c6ff);
      box-shadow: 0 0 18px rgba(0, 150, 255, 0.6);
      transform: scale(1.05);
    }

    /* Reset Button */
    .reset-btn {
      display: inline-block;
      margin-top: 15px;
      padding: 10px 18px;
      border-radius: 8px;
      border: none;
      background: linear-gradient(90deg, #ff4b2b, #ff416c);
      color: #fff;
      font-weight: 600;
      cursor: pointer;
      transition: 0.3s;
    }
    .reset-btn:hover {
      background: linear-gradient(90deg, #ff416c, #ff4b2b);
      transform: scale(1.05);
    }

    .loader {
      margin: 20px auto;
      border: 4px solid rgba(255,255,255,0.2);
      border-top: 4px solid #9b5de5;
      border-radius: 50%;
      width: 40px;
      height: 40px;
      animation: spin 1s linear infinite;
    }

    @keyframes spin { 0% { transform: rotate(0deg);} 100% { transform: rotate(360deg);} }
    @keyframes fadeIn { from {opacity:0;transform:translateY(20px);} to {opacity:1;transform:translateY(0);} }
  </style>
</head>
<body>
  <div class="converter-box">
    <h1>⚡ File Converter</h1>
    <p>Drag & drop your image below or click to upload.</p>

    <div class="drop-zone" id="dropZone">Drop image here or click to upload</div>
    <input type="file" id="fileInput" accept="image/*" hidden>

    <select id="formatSelect">
      <option value="image/jpeg">JPG</option>
      <option value="image/png">PNG</option>
      <option value="image/webp">WEBP</option>
      <option value="image/gif">GIF</option>
      <option value="image/bmp">BMP</option>
      <option value="image/svg+xml">SVG</option>
      <option value="image/tiff">TIFF</option>
      <option value="application/pdf">PDF</option>
    </select>

    <button id="convertBtn">Convert Now</button>
    <div id="preview"></div>
  </div>

  <script>
    const dropZone = document.getElementById("dropZone");
    const fileInput = document.getElementById("fileInput");
    const convertBtn = document.getElementById("convertBtn");
    const preview = document.getElementById("preview");
    let selectedFile = null;

    dropZone.addEventListener("click", () => fileInput.click());
    fileInput.addEventListener("change", (e) => handleFile(e.target.files[0]));

    dropZone.addEventListener("dragover", (e) => {
      e.preventDefault();
      dropZone.classList.add("dragover");
    });
    dropZone.addEventListener("dragleave", () => dropZone.classList.remove("dragover"));
    dropZone.addEventListener("drop", (e) => {
      e.preventDefault();
      dropZone.classList.remove("dragover");
      handleFile(e.dataTransfer.files[0]);
    });

    function handleFile(file) {
      if (!file) return;
      selectedFile = file;
      dropZone.textContent = `✅ ${file.name} selected`;
      preview.innerHTML = "";
    }

    convertBtn.addEventListener("click", async () => {
      if (!selectedFile) {
        alert("Please upload a file first!");
        return;
      }

      const format = document.getElementById("formatSelect").value;
      preview.innerHTML = `<div class='loader'></div><p>Converting...</p>`;

      const reader = new FileReader();
      reader.onload = (e) => {
        const img = new Image();
        img.src = e.target.result;

        img.onload = () => {
          const canvas = document.createElement("canvas");
          const ctx = canvas.getContext("2d");
          canvas.width = img.width || 500;
          canvas.height = img.height || 500;
          ctx.drawImage(img, 0, 0);

          let outputFormat = format.split('/')[1];
          if (outputFormat === "jpeg") outputFormat = "jpg";
          if (outputFormat === "svg+xml") outputFormat = "svg";

          let newDataUrl = canvas.toDataURL(format);

          if (format === "application/pdf") {
            const pdf = new jsPDF();
            pdf.addImage(newDataUrl, "JPEG", 10, 10, 180, 160);
            const pdfUrl = pdf.output("bloburl");
            preview.innerHTML = `
              <a href="${pdfUrl}" download="converted.pdf" class="download-btn">⬇️ Download PDF</a>
              <button class="reset-btn" onclick="resetConverter()">Reset</button>
            `;
            return;
          }

          preview.innerHTML = `
            <img src="${newDataUrl}" alt="Converted Image"><br>
            <a href="${newDataUrl}" download="converted.${outputFormat}" class="download-btn">⬇️ Download File</a>
            <button class="reset-btn" onclick="resetConverter()">Reset</button>
          `;
        };
      };
      reader.readAsDataURL(selectedFile);
    });

    function resetConverter() {
      selectedFile = null;
      fileInput.value = "";
      dropZone.textContent = "Drop image here or click to upload";
      preview.innerHTML = "";
    }
  </script>

  <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
</body>
</html>
