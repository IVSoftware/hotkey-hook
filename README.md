As I understand it, you have these requirements:
- Control-F makes a new form
- Regardless of which Form (or control) has the focus currently, the hotkey should still work.
- The hotkey is local to the application.

Implementing `IMessageFilter` should be effective in achieving these objectives:

[![screenshot][1]][1]

    public partial class MainForm : Form, IMessageFilter
    {
        public MainForm()
        {
            InitializeComponent();
            Application.AddMessageFilter(this);
            Disposed += (sender, e) =>Application.RemoveMessageFilter(this);
        }
        const int WM_KEYDOWN = 0x0100;
        public bool PreFilterMessage(ref Message m)
        {
            switch (m.Msg)
            {
                case WM_KEYDOWN:
                    switch((Keys)m.WParam | ModifierKeys)
                    {
                        case Keys.Control | Keys.F:
                            onNewForm();
                            break;
                    }
                    break;
            }
            return false;
        }

        int _count = 0;
        private void onNewForm()
        {
            char c = (char)(_count + 'A');
            int dim = ++_count * SystemInformation.CaptionHeight;
            new TextBoxForm
            {
                Name = $"textBoxForm{c}",
                Text = $"TextBoxForm {c}",
                Size = this.Size,
                StartPosition= FormStartPosition.Manual,
                Location = new Point(Left + dim, Top + dim),
            }.Show(this);
        }
    }


  [1]: https://i.stack.imgur.com/FZx8y.png