/* QAcademy Teacher Access — Pages Frontend (DEV) */

(function(){
  const QA = (window.QA = window.QA || {});

  // ===== REQUIRED =====
  QA.TOKEN_KEY = "qa_token";

  // Teacher Assess WebApp (Apps Script) — backend for Pages
  QA.BACKEND_URL = "https://script.google.com/macros/s/AKfycbz54M_jWoRq4cOC2gUQeRhd55AeEJ5vnELJ0gkBSodO58fr7WGa9jwk6JNwPptf8p8X/exec";

  // Optional: Portal/Blogger “Teacher Access” entrypoint (for users who land here directly)
  // Put your DEV blogger/portal URL here if you want “Go back” links to work nicely.
  QA.PORTAL_TEACHER_HANDOFF_URL = ""; // e.g. "https://portal.qacademynurses.com/p/teacher-handoff.html"

  // ===== storage =====
  QA.getToken = function(){
    try { return localStorage.getItem(QA.TOKEN_KEY) || ""; } catch(e){ return ""; }
  };
  QA.setToken = function(tok){
    try { localStorage.setItem(QA.TOKEN_KEY, String(tok||"")); } catch(e){}
  };
  QA.clearToken = function(){
    try { localStorage.removeItem(QA.TOKEN_KEY); } catch(e){}
  };

  QA.signOut = function(){
    QA.clearToken();
    location.href = "/";
  };

  // ===== url helpers =====
  QA.qs = function(){
    return new URLSearchParams(location.search || "");
  };
  QA.safePath = function(p){
    p = String(p || "").trim();
    if (!p) return "";
    if (!p.startsWith("/")) return "";
    if (p.startsWith("//")) return "";
    if (p.includes("..")) return "";
    return p;
  };

  // ===== network (NO preflight) =====
  QA.postForm = async function(payload){
    const body = new URLSearchParams(payload || {});
    const res = await fetch(QA.BACKEND_URL, {
      method: "POST",
      headers: { "Content-Type": "application/x-www-form-urlencoded" },
      body
    });

    // Apps Script returns JSON text output (status always 200 usually)
    const text = await res.text();
    let obj = null;
    try { obj = JSON.parse(text); } catch(e){ obj = null; }

    if (!obj) {
      return { ok:false, error:"bad_json", raw:text };
    }
    return obj;
  };

  // ===== backend calls =====
  QA.exchangeHandoff = async function(code){
    return await QA.postForm({
      action: "auth.handoff.exchange",
      code: String(code || "").trim(),
      used_by: "pages",
      ua: navigator.userAgent
    });
  };

  QA.teacherVerify = async function(token){
    return await QA.postForm({
      action: "teacher.verify",
      token: String(token || "").trim()
    });
  };

  QA.bankList = async function(params){
    const token = QA.getToken();
    const payload = Object.assign({ action:"bank.items.list", token }, (params || {}));
    return await QA.postForm(payload);
  };

  QA.bankGet = async function(bank_item_id){
    const token = QA.getToken();
    return await QA.postForm({ action:"bank.item.get", token, bank_item_id });
  };

  QA.bankCreate = async function(fields){
    const token = QA.getToken();
    return await QA.postForm(Object.assign({ action:"bank.item.create", token }, fields || {}));
  };

  QA.bankUpdate = async function(fields){
    const token = QA.getToken();
    return await QA.postForm(Object.assign({ action:"bank.item.update", token }, fields || {}));
  };

  QA.bankDelete = async function(bank_item_id){
    const token = QA.getToken();
    return await QA.postForm({ action:"bank.item.delete", token, bank_item_id });
  };

  // ===== routing =====
  QA.routeByRole = async function({ preferReturnTo }){
    const rt = QA.safePath(preferReturnTo);

    const token = QA.getToken();
    if (!token) {
      // No token on Pages domain → must come via handoff
      const u = new URL("/auth/login.html", location.origin);
      if (rt) u.searchParams.set("return_to", rt);
      location.href = u.toString();
      return;
    }

    const vr = await QA.teacherVerify(token);
    if (vr && vr.ok && vr.teacher_ok) {
      location.href = rt && rt.startsWith("/teacher/") ? rt : "/teacher/";
      return;
    }

    // Not teacher (or not approved) → student
    location.href = rt && rt.startsWith("/student/") ? rt : "/student/";
  };

  QA.requireTeacherOrRoute = async function(){
    const tok = QA.getToken();
    if (!tok) return QA.routeByRole({ preferReturnTo: location.pathname });

    const vr = await QA.teacherVerify(tok);
    if (vr && vr.ok && vr.teacher_ok) return vr;

    // token invalid or not teacher → student
    return QA.routeByRole({ preferReturnTo: "/student/" });
  };

  QA.requireTokenOrBridge = function(returnTo){
    const tok = QA.getToken();
    if (tok) return true;

    const u = new URL("/auth/login.html", location.origin);
    const rt = QA.safePath(returnTo || location.pathname);
    if (rt) u.searchParams.set("return_to", rt);
    location.href = u.toString();
    return false;
  };

})();
